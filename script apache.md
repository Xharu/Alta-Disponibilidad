# 🖥️ Script Interactivo de Administración de Apache2 (`apache.sh`)

Para la demostración de **alta disponibilidad con Keepalived + Apache2**, se creó un **script interactivo** que permite administrar Apache2 de manera rápida y estandarizada en los 4 servidores.

---

## 📌 Objetivo del Script

El script `apache.sh` permite:

- Iniciar, detener y reiniciar Apache2.
- Recargar la configuración sin interrumpir el servicio.
- Habilitar o deshabilitar Apache2 al inicio del sistema.
- Visualizar el estado y los logs en tiempo real.
- Verificar la configuración de Apache2 (`apache2ctl -t`).

**Motivo:** Facilita la simulación de fallos para probar la tolerancia a fallos de Keepalived y garantiza que todos los nodos tengan un control estandarizado de Apache2.

---

## 🛠️ Creación e Instalación del Script

1. **Crear el archivo del script**

    ```
    sudo nano apache.sh
    ```

2. **Pegar dentro el siguiente contenido**

    ```
    #!/bin/bash

    SERVICE="apache2"

    # Verificar permisos de root
    if [[ $EUID -ne 0 ]]; then
        echo "❌ Debes ejecutar este script como root (sudo)."
        exit 1
    fi

    # Funciones
    start_apache() { systemctl start $SERVICE; echo "✔ Apache iniciado."; sleep 1; }
    stop_apache() { systemctl stop $SERVICE; echo "✔ Apache detenido."; sleep 1; }
    restart_apache() { systemctl restart $SERVICE; echo "✔ Apache reiniciado."; sleep 1; }
    reload_apache() { systemctl reload $SERVICE; echo "✔ Configuración de Apache recargada."; sleep 1; }
    status_apache() { systemctl status $SERVICE --no-pager; }
    enable_apache() { systemctl enable $SERVICE; echo "✔ Apache habilitado al inicio."; sleep 1; }
    disable_apache() { systemctl disable $SERVICE; echo "✔ Apache deshabilitado del arranque."; sleep 1; }
    logs_apache() { echo "📜 Mostrando logs de Apache (CTRL+C para salir)"; journalctl -u $SERVICE -f; }
    check_config() { echo "🔍 Verificando configuración de Apache..."; apache2ctl -t; sleep 1; }

    # Menú interactivo
    while true; do
        clear
        echo "=========================================="
        echo "     🖥️  PANEL DE CONTROL - APACHE2"
        echo "=========================================="
        echo "1) Iniciar Apache"
        echo "2) Detener Apache"
        echo "3) Reiniciar Apache"
        echo "4) Recargar configuración"
        echo "5) Ver estado"
        echo "6) Habilitar en el arranque"
        echo "7) Deshabilitar del arranque"
        echo "8) Ver logs en tiempo real"
        echo "9) Verificar configuración (apache2ctl -t)"
        echo "0) Salir"
        echo "------------------------------------------"
        read -p "Seleccione una opción: " opcion

        case $opcion in
            1) start_apache ;;
            2) stop_apache ;;
            3) restart_apache ;;
            4) reload_apache ;;
            5) status_apache; read -p "Presione Enter para continuar..." ;;
            6) enable_apache ;;
            7) disable_apache ;;
            8) logs_apache ;;
            9) check_config ;;
            0) echo "Saliendo..."; exit 0 ;;
            *) echo "❌ Opción inválida."; sleep 1 ;;
        esac
    done
    ```

3. **Guardar y cerrar el archivo**

    En `nano`:

    - `Ctrl + O`, Enter para guardar.  
    - `Ctrl + X` para salir.

4. **Dar permisos de ejecución**

    ```
    sudo chmod +x apache.sh
    ```

5. **Ejecutar el script**

    ```
    sudo apache.sh
    ```

---

## 🧪 Uso del Script en Pruebas de Alta Disponibilidad (HA)
<img width="546" height="357" alt="image" src="https://github.com/user-attachments/assets/26daea81-2054-4910-89ed-8a14d1d481ab" />

- **Detener Apache en el MASTER para simular caída:**
    - Ejecutar el script y elegir:
        - Opción `2) Detener Apache`.

- **Observar la conmutación al BACKUP:**
    - Keepalived detectará la caída y moverá la IP virtual al nodo BACKUP correspondiente.

- **Restablecer el servicio en el MASTER:**
    - Ejecutar el script y elegir:
        - Opción `3) Reiniciar Apache` o `1) Iniciar Apache`.

- **Verificar el estado de Apache y comportamiento de HA:**
    - Opción `5) Ver estado`.
    - Opción `8) Ver logs en tiempo real`.

**Motivo:**  
Permite demostrar de forma práctica la **tolerancia a fallos**, observando cómo la IP virtual pasa del nodo MASTER a los nodos BACKUP cuando el servicio Apache2 falla o se detiene manualmente.

---
