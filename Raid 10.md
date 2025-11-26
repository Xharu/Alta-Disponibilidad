# 🛡️ Configuración de RAID 10 en Linux

Este documento detalla el procedimiento paso a paso para configurar un arreglo **RAID 10** utilizando la herramienta `mdadm` en un entorno Linux (Ubuntu/Debian).

## 📋 Requisitos Previos
* Tener acceso de superusuario (`sudo`).
* 4 discos disponibles (en este ejemplo: `sdb`, `sdc`, `sdd`, `sde`).

---

## 🚀 Pasos de Instalación

1. **Preparación del Sistema:**

    - Primero, actualizamos los repositorios e instalamos la utilidad necesaria para gestionar dispositivos RAID.
    - Actualizamos las listas de paquetes:
        ```bash
        sudo apt update
        ```

    - Instalamos la herramienta mdadm:
        ```bash
        sudo apt install mdadm -y
        ```

2. **Creación del dispositivo RAID 10:**

    - Creamos el dispositivo virtual `/dev/md0` utilizando los 4 discos físicos. El nivel 10 ofrece alta velocidad y redundancia.
        ```bash
        sudo mdadm --create --verbose /dev/md0 --level=10 --raid-devices=4 /dev/sdb /dev/sdc /dev/sdd /dev/sde
        ```

3. **Persistencia de la configuración:**

    - **Guardar la configuración:** Es crítico guardar la configuración en `mdadm.conf` y actualizar el `initramfs`. Si se omite este paso, el RAID podría perderse o cambiar de nombre al reiniciar.
        ```bash
        sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
        ```

    - Actualizamos el entorno de inicio:
        ```bash
        sudo update-initramfs -u
        ```

4. **Formateo y Montaje:**

    - Damos formato `ext4` al nuevo volumen:
        ```bash
        sudo mkfs.ext4 /dev/md0
        ```

    - Creamos el directorio de montaje:
        ```bash
        sudo mkdir -p /mnt/raid10
        ```

    - Montamos el volumen RAID en el directorio creado:
        ```bash
        sudo mount /dev/md0 /mnt/raid10
        ```

5. **Verificación del Estado:**

    - Verificamos el espacio montado para asegurar que el sistema lo reconoce:
        ```bash
        df -h | grep md0
        ```

    - Vemos los detalles profundos del RAID (estado de los discos, sincronización, etc.):
        ```bash
        sudo mdadm --detail /dev/md0
        ```
