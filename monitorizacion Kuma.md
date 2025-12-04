# Instalación y Configuración de Uptime Kuma mediante Docker en el Servidor de Proxy

Detallamos el procedimiento para implementar un sistema de **monitoreo centralizado** con **Uptime Kuma**, ejecutado en Docker sobre el servidor Proxy, para supervisar el estado de los servidores del clúster y la IP virtual de alta disponibilidad.

---

## 📘 Propósito del Procedimiento

- Implementar una plataforma gráfica accesible desde la LAN para monitorear en tiempo real:
  - Servidores: `proxy`, `angel`, `brayan`, `eli`.
  - Servicios: web (HTTP/HTTPS), DNS, Apache.
  - Disponibilidad de la IP virtual de alta disponibilidad: `192.168.100.100`.

---

## 📋 Requisitos Previos

* Servidor de infraestructura basado en Ubuntu.
* Acceso como `sudo`.
* Conectividad con el DNS interno: `192.168.100.6`.
* Paquetes necesarios: `docker`, `docker-compose`.

---

## 🚀 Instalación de Docker en el Servidor de Infraestructura

1. **Instalar Docker mediante script oficial**

    ```
    curl -fsSL https://get.docker.com | sh
    ```

2. **Instalar Docker Compose**

    ```
    sudo apt install -y docker-compose
    ```

3. **Habilitar e iniciar el servicio Docker**

    ```
    sudo systemctl enable --now docker
    ```

---

## 🚀 Levantar el Servicio de Uptime Kuma

1. **Ejecutar el contenedor de Uptime Kuma**

    - Lanzamos el contenedor y configuramos reinicio automático:

    ```
    sudo docker run -d --restart=always \
      -p 3001:3001 \
      -v uptime-kuma:/app/data \
      --name uptime-kuma \
      louislam/uptime-kuma:2
    ```

    - Explicación rápida:
      - `-d`: modo demonio (en segundo plano).
      - `--restart=always`: se inicia tras reinicios del sistema.
      - `-p 3001:3001`: expone Uptime Kuma en el puerto 3001.
      - `-v uptime-kuma:/app/data`: volumen persistente para datos.

2. **Comprobar que el contenedor está en ejecución**

    ```
    sudo docker ps
    ```

    - Debe aparecer un contenedor llamado `uptime-kuma` con estado `Up`.

---

## 🌐 Acceso al Panel de Monitoreo dentro de la LAN

El servicio de Uptime Kuma queda disponible en:

- Por nombre de host (DNS interno):
  - `http://infraestructura.usfx.edu:3001`

- Por dirección IP:
  - `http://192.168.100.2:3001`

Desde este panel se pueden añadir monitores para:
- IP virtual `192.168.100.100`.
- Hosts: `angel.usfx.edu`, `brayan.usfx.edu`, `eli.usfx.edu`, etc.
- Servicios HTTP/HTTPS de los servidores del clúster.

---

## 🔧 Configuración del DNS Interno para Docker

Para que Uptime Kuma pueda resolver correctamente los dominios internos (`angel.usfx.edu`, `brayan.usfx.edu`, `www.keepalived.usfx.edu`, etc.), configuramos Docker para usar el DNS institucional.

1. **Editar la configuración del demonio Docker**

    ```
    sudo nano /etc/docker/daemon.json
    ```

2. **Contenido del archivo**

    ```
    {
      "dns": ["192.168.100.6"]
    }
    ```

3. **Reiniciar Docker para aplicar cambios**

    ```
    sudo systemctl restart docker
    ```

A partir de este punto, los contenedores (incluido Uptime Kuma) usarán el DNS `192.168.100.6` para resolver los nombres internos de la infraestructura.

---
<img width="1913" height="890" alt="Captura de pantalla 2025-12-03 192437" src="https://github.com/user-attachments/assets/e72e45e9-b66f-44c9-bbcb-eaebc82b4e2e" />

