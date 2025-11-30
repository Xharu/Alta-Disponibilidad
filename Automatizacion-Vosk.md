# 🎙️ Control de Apache con la Voz utilizando Vosk + Hotword

Esta parte lo realizaremos en el servidor proxy usando **Vosk**, que es un **asistente por voz**, preparar el audio y permitir el **control de Apache mediante comandos hablados** en Linux.

---

## 📋 Requisitos Previos

* Acceso de superusuario (`sudo`).
* Micrófono funcional.
* Conexión a internet.
* Sistema Linux basado en Ubuntu.

---

## 🚀 Pasos de Instalación

1. **Configuración del Audio del Sistema**

    - Actualizamos repositorios e instalamos ALSA:
        ```
        sudo apt update
        sudo apt install alsa-utils -y
        ```

    - Abrimos el mezclador de audio:
        ```
        alsamixer
        ```

    - Dentro de `alsamixer`:
      <img width="1351" height="702" alt="Captura de pantalla 2025-11-30 013954" src="https://github.com/user-attachments/assets/9ffa43b6-68ba-4b16-a674-53fc2d33e83f" />

        - `F6`: seleccionar la tarjeta de sonido correcta.
        - `F5`: mostrar Playback y Capture.
        - Usar flechas para ajustar “Mic” o “Capture”, subir volumen y asegurarse que aparezca `OO` (no `MM`).

    - Probamos el micrófono:
        ```
        arecord -f S16_LE -r 16000 -d 5 test16.wav
        aplay test16.wav
        ```

2. **Instalación de Python, PortAudio y creación del entorno virtual**

    - Instalamos dependencias:
        ```
        sudo apt update
        sudo apt install python3 python3-venv python3-pip -y
        sudo apt install portaudio19-dev -y
        ```

    - Creamos el entorno virtual para Vosk:
        ```
        python3 -m venv envvosk
        source envvosk/bin/activate
        pip install --upgrade pip
        pip install vosk sounddevice
        ```

3. **Descarga del Modelo de Vosk en Español**

    - Creamos el directorio y descargamos el modelo:
        ```
        cd ~
        mkdir -p modelos_vosk
        cd modelos_vosk
        wget https://alphacephei.com/vosk/models/vosk-model-small-es-0.42.zip
        unzip vosk-model-small-es-0.42.zip
        ```

4. **Asignar Permisos para Controlar Apache sin Contraseña**

    - Abrimos el archivo sudoers:
        ```
        sudo visudo
        ```

    - Añadimos al final (ajustar usuario si es necesario):
        ```
        angel ALL=(ALL) NOPASSWD:/bin/systemctl start apache2,/bin/systemctl stop apache2
        ```

5. **Copiar y Preparar el Sonido de Confirmación**

    - Desde Windows, transferimos el archivo de sonido:
        ```
        scp "E:\2-2025\SIS313\final\short-raccoon-sound.mp3" angel@192.168.100.2:~
        ```

    - Instalamos un reproductor MP3:
        ```
        sudo apt update
        sudo apt install mpg123 -y
        ```

6. **Crear el Script del Asistente por Voz**

    - Creamos el archivo Python:
        ```
        nano vosk_apache_hotword.py
        ```

    - Pegamos el siguiente contenido:
        ```
        import queue
        import sounddevice as sd
        import vosk
        import json
        import subprocess
        import unicodedata
        import time
        import os

        MODEL_PATH = "/home/angel/modelos_vosk/vosk-model-small-es-0.42"
        SONIDO_MAPACHE = "/home/angel/short-raccoon-sound.mp3"
        LOG_PATH = "/home/angel/vosk_apache.log"

        PALABRAS_CLAVE = ["con", "com"]

        PALABRAS_ENCENDER = [
            "enciende apache",
            "prende apache",
            "prender apache",
            "encender apache",
        ]

        PALABRAS_APAGAR = [
            "apaga apache",
            "apagar apache",
            "apague apache",
        ]

        COOLDOWN_HOTWORD = 2.0  # segundos entre activaciones

        def normalizar(frase):
            frase = frase.lower().strip()
            frase = ''.join(
                c for c in unicodedata.normalize('NFD', frase)
                if unicodedata.category(c) != 'Mn'
            )
            return frase

        def log_event(msg):
            marca = time.strftime('%Y-%m-%d %H:%M:%S')
            linea = f"[{marca}] {msg}\n"
            print(linea, end="")
            try:
                with open(LOG_PATH, "a") as f:
                    f.write(linea)
            except Exception:
                pass

        def sonar_mapache():
            try:
                os.system(f"mpg123 {SONIDO_MAPACHE} >/dev/null 2>&1")
            except Exception as e:
                log_event(f"No se pudo reproducir el sonido: {e}")

        def ejecutar_accion(texto):
            t = normalizar(texto)

            if any(normalizar(p) in t for p in PALABRAS_ENCENDER):
                log_event("Comando reconocido: ENCENDER APACHE")
                res = subprocess.run(
                    ["sudo", "systemctl", "start", "apache2"],
                    capture_output=True, text=True
                )
                log_event(f"systemctl start apache2 -> rc={res.returncode}, out={res.stdout.strip()}, err={res.stderr.strip()}")
            elif any(normalizar(p) in t for p in PALABRAS_APAGAR):
                log_event("Comando reconocido: APAGAR APACHE")
                res = subprocess.run(
                    ["sudo", "systemctl", "stop", "apache2"],
                    capture_output=True, text=True
                )
                log_event(f"systemctl stop apache2 -> rc={res.returncode}, out={res.stdout.strip()}, err={res.stderr.strip()}")
            else:
                log_event(f"Comando no reconocido: '{texto}'")

        model = vosk.Model(MODEL_PATH)
        q = queue.Queue()

        def callback(indata, frames, time_info, status):
            q.put(bytes(indata))

        def escuchar_una_frase(timeout=None):
            rec = vosk.KaldiRecognizer(model, 16000)
            inicio = time.time()
            while True:
                if timeout and time.time() - inicio > timeout:
                    return None
                data = q.get()
                if rec.AcceptWaveform(data):
                    res = json.loads(rec.Result())
                    texto = res.get("text", "")
                    if texto:
                        return texto

        with sd.RawInputStream(samplerate=16000, blocksize=8000,
                               dtype='int16', channels=1, callback=callback):
            log_event("Sistema listo. Di la palabra clave: 'con' o 'com' (Ctrl+C para salir).")
            ultimo_hotword = 0.0
            while True:
                texto = escuchar_una_frase()
                if not texto:
                    continue
                log_event(f"Texto escuchado: {texto}")
                tnorm = normalizar(texto)

                ahora = time.time()
                if ahora - ultimo_hotword < COOLDOWN_HOTWORD:
                    continue

                if any(normalizar(p) in tnorm for p in PALABRAS_CLAVE):
                    ultimo_hotword = ahora
                    log_event("Palabra clave detectada, sonando mapache...")
                    sonar_mapache()
                    log_event("Escuchando comando (5 segundos)...")
                    comando = escuchar_una_frase(timeout=5)
                    if comando:
                        log_event(f"Comando detectado: {comando}")
                        ejecutar_accion(comando)
                    else:
                        log_event("No se detectó comando a tiempo.")
        ```

7. **Ejecutar el Asistente de Voz**

    - Con el entorno virtual activado (`source envvosk/bin/activate`), ejecutamos:
        ```
        python vosk_apache_hotword.py
        ```

---

## 🧪 Resultado Ejemplo (Log)

Ejemplo de salida en `vosk_apache.log`:

- [2025-11-30 07:03:17] Sistema listo. Di la palabra clave: 'con' o 'com' (Ctrl+C para salir).
- [2025-11-30 07:03:22] Texto escuchado: com
- [2025-11-30 07:03:22] Palabra clave detectada, sonando mapache...
- [2025-11-30 07:03:24] Escuchando comando (5 segundos)...
- [2025-11-30 07:03:28] Comando detectado: enciende apache
- [2025-11-30 07:03:28] Comando reconocido: ENCENDER APACHE
- [2025-11-30 07:03:28] systemctl start apache2 -> rc=0, out=, err=

