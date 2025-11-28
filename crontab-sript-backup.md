# 💾 Backup Automático con Cron y Rsync

Este documento detalla el procedimiento paso a paso para implementar un sistema de **respaldo automático** de los archivos web de todos los servidores del clúster utilizando un **script Bash**, **rsync** y tareas programadas con **cron**.

---

## 📋 Requisitos Previos

* Tener acceso de superusuario (`sudo`) o al menos al usuario `angel`.
* Conectividad SSH desde el servidor de backups hacia:
  * `192.168.100.2` (infraestructura)
  * `192.168.100.3` (angel)
  * `192.168.100.4` (brayan)
  * `192.168.100.5` (eli)
* Directorios web en:
  * `/srv/ftp/websites/rootwebservers/<nombre>/public_html/`

---

## 🚀 Pasos de Configuración

1. **Instalación de Rsync**

    - Actualizamos repositorios e instalamos `rsync`:
        ```
        sudo apt update
        sudo apt install rsync -y
        ```

    - Propósito:
        - `rsync` permite sincronizar/copiar archivos preservando permisos, fechas y estructura.

2. **Preparar directorios de respaldo**

    - Creamos el directorio base para guardar todos los backups:
        ```
        mkdir -p /home/angel/backups
        ```

    - Estructura esperada (ejemplo):
        ```
        /home/angel/backups/angel/20-11-2025_08-00/
        /home/angel/backups/brayan/20-11-2025_08-00/
        /home/angel/backups/eli/20-11-2025_08-00/
        ...
        ```
       <img width="459" height="95" alt="image" src="https://github.com/user-attachments/assets/612013fc-f87e-4b90-bd78-6f7f52630265" />

3. **Crear el script de backup**

    - Creamos el archivo:
        ```
        nano backup_automatico.sh
        ```

    - Contenido del script:
        ```
        #!/bin/bash

        # Captura fecha y hora
        FECHA=$(date +"%d-%m-%Y_%H-%M")

        DIRECTORIO_BASE="/home/angel/backups"

        echo "INICIANDO BACKUP: $FECHA"

        # Función para realizar el backup
        realizar_backup() {
            IP_SERVIDOR=$1
            NOMBRE=$2

            ORIGEN="angel@$IP_SERVIDOR:/srv/ftp/websites/rootwebservers/$NOMBRE/public_html/"
            DESTINO="$DIRECTORIO_BASE/$NOMBRE/$FECHA/"

            mkdir -p "$DESTINO"
            rsync -avz "$ORIGEN" "$DESTINO"

            if [ $? -eq 0 ]; then
                echo "Backup de $NOMBRE completado exitosamente."
            else
                echo "ERROR al respaldar $NOMBRE."
            fi
        }

        # Ejecución para cada servidor
        realizar_backup "192.168.100.2" "infraestructura"
        realizar_backup "192.168.100.3" "angel"
        realizar_backup "192.168.100.4" "brayan"
        realizar_backup "192.168.100.5" "eli"

        echo "PROCESO TERMINADO."
        ```

    - Guardar y cerrar (`Ctrl + O`, Enter, `Ctrl + X`).

    - Dar permisos de ejecución:
        ```
        chmod +x backup_automatico.sh
        ```

    - Explicación breve:
        - `FECHA`: define una marca de tiempo para cada ejecución.
        - `realizar_backup IP NOMBRE`: copia el `public_html` de cada servidor a una carpeta con fecha.
        - `rsync -avz`:
          - `-a`: modo archivo (permisos, fechas, recursivo).
          - `-v`: modo detallado.
          - `-z`: compresión durante la transferencia.

4. **Automatización con Cron**

    - Editamos el crontab del usuario `angel`:
        ```
        crontab -e
        ```

    - Agregamos la línea:
        ```
        # Backup automático diario a las 08:00 AM
        0 8 * * * /home/angel/backup_automatico.sh >> /var/log/mis_backups.log 2>&1
        ```

    - Explicación:
        - `0 8 * * *`: ejecuta el script todos los días a las 08:00.
        - `>> /var/log/mis_backups.log 2>&1`: guarda salida y errores en el log.

---

## 🔄 Verificación del Funcionamiento

1. **Probar el script manualmente**

    ```
    sudo backup_automatico.sh
    ```

2. **Comprobar estructura de backups**

    - Verificar que se generaron carpetas con la fecha:
        ```
        ls /home/angel/backups
        ls /home/angel/backups/angel
        ```

3. **Revisar el log de ejecución**

    ```
    cat /var/log/mis_backups.log
    ```

4. **Verificar contenido copiado**

    - Confirmar que los archivos de los DocumentRoot están respaldados para cada servidor:
        ```
        ls /home/angel/backups/angel/<FECHA>/
        ls /home/angel/backups/brayan/<FECHA>/
        ```

---

## ✅ Observaciones Finales

- Cada servidor mantiene su **DocumentRoot aislado** y los respaldos respetan esa estructura.
- La automatización con `cron` evita intervención manual diaria.
- `rsync` permite respaldos eficientes preservando estructura y permisos.
- Se recomienda:
  - Monitorizar regularmente `/var/log/mis_backups.log`.
  - Asegurar suficiente espacio en `/home/angel/backups/`.
  - Considerar copiar estos backups a un **disco externo o NAS** para mayor redundancia.

---
