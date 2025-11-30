# 🗄️ Configuración de Backups Automáticos con Cron + Rsync + SSH Keys

Se detalla el procedimiento para implementar un sistema de **backups automáticos**, ejecutado a través de **cron**, utilizando **rsync** para sincronizar datos desde varios servidores y **llaves SSH** para evitar solicitudes de contraseña.

---

## 📋 Requisitos Previos

* Acceso de superusuario (`sudo`).
* Conectividad SSH hacia los servidores a respaldar.
* Directorio de destino de respaldos: `/home/angel/backups`
* Rutas de origen creadas:
  * `/srv/ftp/websites/rootwebservers/superwebmaster/*/public_html/`

---

## 🚀 Instalación de Paquetes Necesarios

1. **Actualizar repositorios**

    ```
    sudo apt update
    ```

2. **Instalar rsync (herramienta principal del backup)**

    ```
    sudo apt install rsync -y
    ```

3. **Instalar cron (servicio que ejecutará los backups automáticamente)**

    ```
    sudo apt install cron -y
    ```

---

## 🔐 Configuración de Acceso SSH sin Contraseña

Para que `rsync` pueda ejecutarse automáticamente desde `cron`, se configura autenticación SSH por llaves (sin contraseña).

1. **Generar las llaves SSH**

    ```
    ssh-keygen -t rsa -b 4096
    ```

    - Responder:
      - Ubicación: `ENTER` (usar la ruta por defecto `~/.ssh/id_rsa`).
      - Passphrase: `ENTER` (dejar vacío para no pedir contraseña).

    - Esto genera:
      - Llave privada → `~/.ssh/id_rsa`
      - Llave pública → `~/.ssh/id_rsa.pub`

2. **Copiar la llave pública a cada servidor**

    ```
    ssh-copy-id proxy@192.168.100.2
    ssh-copy-id angel@192.168.100.3
    ssh-copy-id brayann@192.168.100.4
    ssh-copy-id eli@192.168.100.5
    ```

3. **Verificar acceso sin contraseña**

    ```
    ssh angel@192.168.100.2
    ```

    - Debe ingresar sin solicitar password.

---

## 📅 Programación de Tareas Automáticas con Cron

1. **Abrir el archivo de cron del usuario**

    ```
    crontab -e
    ```

2. **Agregar las ejecuciones automáticas**

    ```
    # Mañana (08:00)
    00 08 * * * /scripts/backup_automatico.sh >> /var/log/mis_backups.log 2>&1

    # Tarde (14:00)
    00 14 * * * /scripts/backup_automatico.sh >> /var/log/mis_backups.log 2>&1

    # Noche (22:00)
    00 22 * * * /scripts/backup_automatico.sh >> /var/log/mis_backups.log 2>&1
    ```

---

## 📝 Creación del Script de Respaldo

1. **Crear el script**

    ```
    nano backup_automatico.sh
    ```

2. **Insertar el siguiente contenido**

    ```
    #!/bin/bash

    # --- CONFIGURACIÓN ---
    FECHA=$(date +"%d-%m-%Y_%H-%M")
    DIRECTORIO_BASE="/home/angel/backups"

    echo "=========================================="
    echo " INICIANDO BACKUP: $FECHA "
    echo "=========================================="

    realizar_backup() {
        IP_SERVIDOR=$1
        NOMBRE=$2

        ORIGEN="angel@$IP_SERVIDOR:/srv/ftp/websites/rootwebservers/superwebmaster/$NOMBRE/public_html/"
        DESTINO="$DIRECTORIO_BASE/$NOMBRE/$FECHA/"

        echo "[+] Procesando: $NOMBRE ($IP_SERVIDOR)..."

        mkdir -p "$DESTINO"

        rsync -avz "$ORIGEN" "$DESTINO"

        if [ $? -eq 0 ]; then
            echo "✅ Backup de $NOMBRE completado exitosamente."
        else
            echo "❌ ERROR al respaldar $NOMBRE."
        fi
        echo "------------------------------------------"
    }

    realizar_backup "192.168.100.2" "infraestructura"
    realizar_backup "192.168.100.3" "angel"
    realizar_backup "192.168.100.4" "brayan"
    realizar_backup "192.168.100.5" "eli"

    echo "PROCESO TERMINADO."
    ```

3. **Dar permisos de ejecución**

    ```
    sudo chmod +x backup_automatico.sh
    ```

---

## ▶️ Ejecución Manual (Backup de Emergencia)

    ./backup_automatico.sh
---

## ✔️ Resultado

Con este procedimiento:

- `cron` ejecutará automáticamente los backups **3 veces al día**.
- `rsync` sincronizará las carpetas `public_html` de cada servidor remoto.
- Todas las conexiones se realizarán mediante **llaves SSH**, sin pedir contraseña.
- Los respaldos quedarán organizados por **servidor y fecha** dentro de `/home/angel/backups`.

---

## Ejemplos
<img width="844" height="919" alt="image" src="https://github.com/user-attachments/assets/ddccdf25-cf4f-4411-98b7-64409530ba04" />



