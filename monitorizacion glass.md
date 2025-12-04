# 🌐 Instalación y Configuración de Glass para Administración Web de DHCP (Servidor DHCP–DNS)

Detallamos el procedimiento para instalar y configurar **Glass**, una interfaz web moderna para administrar y visualizar en tiempo real la configuración y los leases del servidor **ISC DHCP**, sobre un servidor que ya tiene **DHCP + DNS (BIND9)** configurado.

---

## 📘 Propósito del Procedimiento

- Habilitar una plataforma web de administración que permita:
  - Visualizar leases activos.
  - Consultar reservas y subredes.
  - Revisar parámetros del servidor DHCP.
- Facilitar la gestión sin necesidad de editar archivos de configuración manualmente.

---

## 📋 Requisitos Previos

* Servidor DHCP–DNS basado en Debian.
* Servicio activo de `isc-dhcp-server`.
* Acceso como `sudo`.
* Conectividad en la red interna.
* Paquetes necesarios: `nodejs`, `npm`, `git`.

---

## 🚀 Instalación de Paquetes Base

1. **Actualizar repositorios**

    ```
    sudo apt update
    ```

2. **Instalar dependencias principales**

    ```
    sudo apt install -y nodejs npm git
    ```

---

## 🚀 Descarga de Glass y Preparación del Entorno

1. **Clonar el repositorio oficial en `/opt`**

    ```
    cd /opt
    sudo git clone https://github.com/Akkadius/glass-isc-dhcp.git
    cd glass-isc-dhcp
    ```

2. **Crear carpeta de logs y asignar permisos**

    ```
    sudo mkdir logs
    sudo chmod u+x ./bin/ -R
    sudo chmod u+x *.sh
    ```

---

## 🚀 Instalación de Dependencias NodeJS y Prueba Inicial

1. **Instalar dependencias NodeJS**

    ```
    sudo npm install
    sudo npm install forever -g
    ```

2. **Iniciar Glass en modo prueba (no permanente)**

    ```
    sudo npm start
    ```

3. **Acceso inicial al panel web**

    - El panel quedará disponible en:
      - `http://192.168.100.6:3000`

---

## 🔧 Ajustar archivo de configuración `glass_config.json`

1. **Copiar archivo de configuración de ejemplo**

    ```
    cd /opt/glass-isc-dhcp
    sudo cp config/glass_config.example.json config/glass_config.json
    ```

2. **Editar el archivo de configuración**

    ```
    sudo nano config/glass_config.json
    ```

3. **Parámetros típicos a ajustar**

    - Ruta del archivo `dhcpd.conf` (por ejemplo: `/etc/dhcp/dhcpd.conf`).
    - Interfaz de escucha.
    - Puerto del panel web (3000 por defecto).
    - Otras opciones según la topología de tu red.

---

## ⚙️ Configurar Glass como Servicio Permanente (systemd)

1. **Crear el archivo de servicio systemd**

    ```
    sudo nano /etc/systemd/system/glass.service
    ```

2. **Contenido recomendado**

    ```
    [Unit]
    Description=Glass ISC DHCP service
    After=network.target isc-dhcp-server.service

    [Service]
    Type=simple
    WorkingDirectory=/opt/glass-isc-dhcp
    ExecStart=/usr/bin/forever --minUptime 10000 --spinSleepTime 10000 -a -o ./logs/glass-process.log -e ./logs/glass-error.log ./bin/www
    Restart=always
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    ```

3. **Aplicar configuración y habilitar el servicio**

    ```
    sudo systemctl daemon-reload
    sudo systemctl enable --now glass.service
    ```

4. **Verificar estado del servicio**

    ```
    sudo systemctl status glass.service
    ```

Si se relizo todo sin errores, Glass quedará ejecutándose de forma permanente y se iniciará automáticamente cada vez que el servidor DHCP–DNS arranque, permitiendo la administración web del servicio DHCP.

---
<img width="1919" height="900" alt="Captura de pantalla 2025-12-03 192449" src="https://github.com/user-attachments/assets/85ecef5f-86e4-46df-8649-e67acc058f5b" />
