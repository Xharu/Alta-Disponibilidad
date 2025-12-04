
# 🦝 Proyecto RaccoonSync: Alta Disponibilidad - Tolerancia a Fallos y Automatizado

Este documento resume la arquitectura, componentes y automatizaciones implementadas en el proyecto **RaccoonSync**, orientado a lograr **alta disponibilidad**, **tolerancia a fallos** y **gestión centralizada automatizada** de servicios en una infraestructura basada en Linux.

---

## 🎓 Información Académica

- **Materia:** Infraestructura, Plataformas Tecnológicas y Redes  
- **Docente:** Quispe Ortega Lucio Marcelo  
- **Integrantes:**
  - Mauricio Torrejon Miguel Ángel  
  - Chungara Choque Elizabeth  
  - Cruz Trujillo Brayann  

---

## 📖 Descripción del Proyecto

El proyecto de **Alta Disponibilidad - Tolerancia a Fallos y Automatizado** tiene como objetivo implementar una arquitectura capaz de garantizar la continuidad del servicio incluso ante fallos inesperados, asegurando que los sistemas permanezcan operativos y minimizando los tiempos de inactividad mediante:

- Tecnologías de **redundancia**.  
- **Balanceo y conmutación** ante fallos.  
- **Recuperación automática** y monitoreo.

---

## 🎯 Objetivos

- Diseñar una infraestructura que mantenga los servicios activos ante fallos de hardware o software.  
- Implementar técnicas de redundancia, replicación, automatización y monitoreo.  
- Evaluar el rendimiento y comportamiento del sistema bajo diferentes escenarios de falla.

---

## 🏗️ Arquitectura del Sistema

- La infraestructura se compone de **6 servidores (físicos/virtualizados)** ejecutando Debian/Ubuntu.  
- El núcleo es un **clúster web** con una **IP Virtual (VIP)** compartida, protegida por Keepalived.

### 📸 Diagrama de Arquitectura

---
<img width="1456" height="622" alt="image" src="https://github.com/user-attachments/assets/3ca0890b-fcfb-4430-9601-7a68de3042ec" />

## ⚙️ Roles de los Servidores

| Servidor          | Rol              | IP Real         | Prioridad Keepalived |
|-------------------|------------------|-----------------|----------------------|
| Infraestructura   | MASTER (Proxy)   | 192.168.100.2   | 100                  |
| Angel             | Esclavo 1        | 192.168.100.3   | 90                   |
| Brayan            | Esclavo 2        | 192.168.100.4   | 80                   |
| Eli               | Esclavo 3        | 192.168.100.5   | 70                   |
| DNS/DHCP          | Servicios de Red | 192.168.100.6   | N/A                  |
| Backup            | Backup Web       | 192.168.100.8   | N/A                  |
| IP Virtual (VIP)  | Acceso Público   | 192.168.100.100 | Compartida           |

---

## 🚀 Módulo 1: Servicios de Red (DHCP + DNS)

Servidor dedicado a la **gestión de direcciones IP** y **resolución de nombres internos**.

### 1.1 Configuración DHCP (isc-dhcp-server)

- **Interfaz:** `ens33`.  
- **Rangos de IP dinámicas:**  
  - 192.168.100.11 – 192.168.100.99  
  - 192.168.100.101 – 192.168.100.254  
- El servidor:
  - Define el dominio local `usfx.edu`.  
  - Asigna automáticamente **DNS (192.168.100.6)** y **gateway** a los clientes.  

### 1.2 Configuración DNS (BIND9)

- **Zona:** `usfx.edu` (tipo **master**).  
- **Registros A:** mapean `angel`, `brayan`, `eli`, `infraestructura`, etc., a sus IP reales.  
- **Alta disponibilidad:**
  - `www.keepalived` y `keepalived` apuntan a la IP Virtual `192.168.100.100`.  
- **Gestión Visual (Glass):**
  - Se instala **Glass**, interfaz web basada en NodeJS para administrar visualmente leases y subredes DHCP en el puerto `3000`.
[DHCP-DNS](./dhcp-dns.md)
---

## 🛡️ Módulo 2: Servidores Web Seguros (Apache + HTTPS)

Cada servidor del clúster ejecuta **Apache2** y se aisla su contenido y acceso.

### 2.1 VirtualHosts y Directorios

- Cada servidor tiene su propio **DocumentRoot** aislado dentro de:  
  - `/srv/ftp/websites/rootwebservers/superwebmaster/<nombre>/public_html`  
- Esta estructura facilita:
  - Gestión mediante usuario **enjaulado (Chroot)**.
  - Separación clara de contenidos entre nodos.
  [VirtualHost-DocumentRoot](./DocumentRoot-VirtualHost.md)

### 2.2 Seguridad SSL/TLS

- Se generan **certificados autofirmados** RSA 2048 bits, válidos por 365 días.  
- Todo el tráfico HTTP (puerto 80) se **redirige forzosamente a HTTPS (443)**.  
- En cada VirtualHost HTTPS se habilita:
  - `SSLEngine on`  
  - `SSLCertificateFile` (.crt)  
  - `SSLCertificateKeyFile` (.key)  
[SSL-TLS](./certificados-SSL-autofirmados-redireccion.md)
---

## 🔄 Módulo 3: Alta Disponibilidad (Keepalived)

Se utiliza el protocolo **VRRP** mediante Keepalived para garantizar la **disponibilidad continua** del servicio web.

- **Lógica Master–Backup:**
  - El servidor **Infraestructura (Proxy)** es el **MASTER**.  
  - Si falla (o se detiene Apache), la IP Virtual `192.168.100.100` migra al siguiente nodo con mayor prioridad:  
    - Angel → Brayan → Eli.

- **Script de salud (`check_apache.sh`):**
  - Keepalived ejecuta cada 2 segundos un script que verifica:
    - `systemctl status apache2`.  
  - Si el script detecta fallo, se aplica un **peso negativo** (por ejemplo `weight -35`), provocando la conmutación del VIP a un nodo sano.
[Keepalived](./keepalived-apache2.md)
---

## 🛠️ Módulo 4: Automatización y Gestión

### 4.1 Backups Automáticos (Cron + Rsync + SSH Keys)

- Sistema de respaldo distribuido usando **Cron** y **Rsync**.  
- **Autenticación:**  
  - Llaves SSH sin contraseña entre los servidores.  
- **Programación:**
  - Backups automáticos a las **08:00**, **14:00** y **22:00**.  
- **Script `backup_automatico.sh`:**
  - Sincroniza las carpetas `public_html` de todos los nodos hacia:  
    - `/home/angel/backups/<servidor>/<fecha>/`  
  - Mantiene respaldos ordenados por fecha y nombre del servidor.
 [backup-cron-Rsync](./Backup-Cron+Rsync.md)

### 4.2 Control por Voz (Vosk)

Sistema innovador para controlar **Apache** mediante comandos de voz en el servidor Proxy.

- **Tecnologías:**
  - Python + **Vosk** (reconocimiento de voz offline) + PortAudio.  
- **Funcionamiento:**
  - Detecta la palabra clave (**hotword**) “con” o “com” y reproduce un sonido (“mapache”) como confirmación.  
  - Luego escucha comandos como:
    - “enciende apache”  
    - “apaga apache”  
  - Ejecuta `systemctl start/stop apache2` mediante `subprocess` y reglas en `sudoers` (sin solicitar contraseña).
 [vosk](./Automatizacion-Vosk.md)

### 4.3 Gestión SFTP Segura (Chroot)

- Usuario dedicado: `superwebmaster`.  
- El usuario está **enjaulado** en: `/srv/ftp/websites/rootwebservers`.  
- Sin acceso a shell interactiva (`nologin`), solo **SFTP** para subir/editar contenido web.
 [sftp-chroot-user](./user-chroot-SFTP.md)

### 4.4 Script de Administración (`apache.sh`)

- Script interactivo en Bash desplegado en todos los nodos para estandarizar la gestión de Apache.  
- Menú que permite:
  - Iniciar, detener, reiniciar Apache.  
  - Ver estado del servicio.  
  - Habilitar/deshabilitar en el arranque.  
  - Ver logs en tiempo real.  
  - Verificar configuración (`apache2ctl -t`).  
- Facilita las **pruebas de caída y recuperación** durante la demostración.
[script apache](./script%20apache.md)
---

## 📊 Módulo 5: Monitoreo (Uptime Kuma)

- Desplegado mediante **Docker** en el servidor de Infraestructura/Proxy.  
- Características:
  - Puerto de acceso: `3001`.  
  - Volumen persistente para guardar la configuración de monitores.  
  - Integrado con el **DNS local (192.168.100.6)** para resolver nombres internos.  
- Función:
  - Monitorea el estado **HTTP/HTTPS** de los 4 servidores web.  
  - Supervisa la **disponibilidad de la IP Virtual `192.168.100.100`**.  
 [monitorizacion Kuma](./monitorizacion%20Kuma.md)
---
## 🪟 Módulo 6: Monitoreo (Glass)

-Para la administración y visualización web del servicio DHCP:  
-Despliegue: Instalación nativa (NodeJS + Git) sobre el servidor DHCP–DNS existente.  

- Configuración:  
  - Puerto: 3000.  
  - Ejecución persistente mediante servicio systemd y gestor de procesos Forever.  
  - Vinculado directamente al archivo de configuración `/etc/dhcp/dhcpd.conf`.
Función: Visualización en tiempo real de leases activos, gestión de reservas/subredes y administración gráfica del servidor ISC DHCP.
[monitorizacion glass](./monitorizacion%20glass.md)
---
## 🧪 Pruebas de Funcionamiento

Pasos típicos para demostrar tolerancia a fallos:

1. Acceder a:  
   - `https://www.keepalived.usfx.edu` (resolviendo a la VIP `192.168.100.100`).  

2. En el servidor MASTER (Infraestructura), ejecutar el script:
   - `sudo apache.sh`  
   - Seleccionar la opción **2) Detener Apache**.  

3. Observar en **Uptime Kuma** y en el navegador:
   - El servicio sigue respondiendo, pero ahora servido desde el **BACKUP 1 (Angel)**.  

4. Reiniciar Apache en el MASTER:
   - La IP Virtual debería volver automáticamente al servidor Infraestructura (**failback**).  

---

## 📄 Conclusiones

- El proyecto **integra múltiples capas de redundancia**:
  - Red (IP Virtual con Keepalived).  
  - Servicios (Apache en clúster).  
  - Datos (backups con Rsync + Cron).  
- La combinación de:
  - **Keepalived** para IP virtual y conmutación.  
  - **Rsync + SSH Keys + Cron** para protección de datos.  
  - **Uptime Kuma y Glass** para monitoreo y administración visual.  
  - **Vosk** para control por voz.  
- Da como resultado una infraestructura:
  - **Robusta**,  
  - **Escalable**,  
  - **Tolerante a fallos**,  
  - Alineada con los principios de **alta disponibilidad**.

Este documento consolida la documentación de nuestro proyecto Final de la materia SIS313, llamado **RACCOONSYNC** y servira como referencia para despliegue, operación y demostración académica.

---
