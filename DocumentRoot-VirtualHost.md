# 🌐 Configuración de DocumentRoot y VirtualHosts por Servidor

Este documento detalla cómo configurar varios **VirtualHosts** en Apache, cada uno con su propio **DocumentRoot** dentro del entorno enjaulado del usuario `superwebmaster`. Esto permite:

- Aislamiento de archivos por servidor.
- Integración con el usuario SFTP en chroot.
- Organización coherente con la arquitectura de **alta disponibilidad**.

---

## 📋 Estructura General

Cada servidor usa una ruta similar para su DocumentRoot:

- Proxy: `/srv/ftp/websites/rootwebservers/superwebmaster/proxy/public_html`
- Angel: `/srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html`
- Brayan: `/srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html`
- Eli: `/srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html`

---

## 🚀 Pasos de Configuración

1. **Crear la estructura de directorios para cada sitio**

    - Creamos las carpetas base (como root o con sudo):

        ```
        sudo mkdir -p /srv/ftp/websites/rootwebservers/superwebmaster/proxy/public_html
        sudo mkdir -p /srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html
        sudo mkdir -p /srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html
        sudo mkdir -p /srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html
        ```

    - (Opcional) Asignar permisos para que `superwebmaster` pueda gestionar el contenido:

        ```
        sudo chown -R superwebmaster:superwebmaster /srv/ftp/websites/rootwebservers/superwebmaster
        ```

2. **Crear archivos de configuración de VirtualHost**

    - Vamos al directorio de sitios disponibles de Apache:

        ```
        cd /etc/apache2/sites-available
        ```

---

## 1️⃣ Servidor Proxy

- **DocumentRoot:**

    ```
    /srv/ftp/websites/rootwebservers/superwebmaster/proxy/public_html
    ```

- **Crear archivo de sitio:**

    ```
    sudo nano /etc/apache2/sites-available/proxy.conf
    ```

- **Contenido de `proxy.conf`:**

    ```
    <VirtualHost *:80>
        ServerName www.proxy.usfx.bo
        ServerAlias www.keepalived.usfx.bo

        DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/proxy/public_html

        <Directory /srv/ftp/websites/rootwebservers/superwebmaster/proxy/public_html>
            Options FollowSymLinks
            AllowOverride All
            Require all granted
            DirectoryIndex index.html
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/proxy-error.log
        CustomLog ${APACHE_LOG_DIR}/proxy-access.log combined
    </VirtualHost>
    ```

---

## 2️⃣ Servidor Angel

- **DocumentRoot:**

    ```
    /srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html
    ```

- **Crear archivo de sitio:**

    ```
    sudo nano /etc/apache2/sites-available/angel.conf
    ```

- **Contenido de `angel.conf`:**

    ```
    <VirtualHost *:80>
        ServerName www.angel.usfx.bo
        ServerAlias www.keepalived.usfx.bo

        DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html

        <Directory /srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html>
            Options FollowSymLinks
            AllowOverride All
            Require all granted
            DirectoryIndex index.html
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/angel-error.log
        CustomLog ${APACHE_LOG_DIR}/angel-access.log combined
    </VirtualHost>
    ```

---

## 3️⃣ Servidor Brayan

- **DocumentRoot:**

    ```
    /srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html
    ```

- **Crear archivo de sitio:**

    ```
    sudo nano /etc/apache2/sites-available/brayan.conf
    ```

- **Contenido de `brayan.conf`:**

    ```
    <VirtualHost *:80>
        ServerName www.brayan.usfx.bo
        ServerAlias www.keepalived.usfx.bo

        DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html

        <Directory /srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html>
            Options FollowSymLinks
            AllowOverride All
            Require all granted
            DirectoryIndex index.html
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/brayan-error.log
        CustomLog ${APACHE_LOG_DIR}/brayan-access.log combined
    </VirtualHost>
    ```

---

## 4️⃣ Servidor Eli

- **DocumentRoot:**

    ```
    /srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html
    ```

- **Crear archivo de sitio:**

    ```
    sudo nano /etc/apache2/sites-available/eli.conf
    ```

- **Contenido de `eli.conf`:**

    ```
    <VirtualHost *:80>
        ServerName www.eli.usfx.bo
        ServerAlias www.keepalived.usfx.bo

        DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html

        <Directory /srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html>
            Options FollowSymLinks
            AllowOverride All
            Require all granted
            DirectoryIndex index.html
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/eli-error.log
        CustomLog ${APACHE_LOG_DIR}/eli-access.log combined
    </VirtualHost>
    ```

---

## 🔄 Activación de los VirtualHosts

Para cada servidor (ejemplos con `angel.conf`, repetir cambiando el nombre):

```
sudo a2ensite proxy.conf
sudo a2ensite angel.conf
sudo a2ensite brayan.conf
sudo a2ensite eli.conf

sudo a2dissite 000-default.conf

sudo apache2ctl configtest
sudo systemctl reload apache2
```
**Explicación:**

- `a2ensite <sitio>.conf` habilita el VirtualHost.
- `a2dissite 000-default.conf` deshabilita el sitio por defecto para evitar conflictos.
- `apache2ctl configtest` verifica que la sintaxis de Apache sea correcta.
- `systemctl reload apache2` aplica los cambios sin detener el servicio.

---

## En Conclusion

- Cada servidor (Proxy, Angel, Brayan, Eli) tiene su propio **DocumentRoot** bajo `superwebmaster`.
- Cada VirtualHost define:
  - Su dominio (`ServerName` y `ServerAlias`).
  - Su ruta de documentos (`DocumentRoot`).
  - Sus permisos y opciones de directorio.
  - Sus propios logs de acceso y error.
- Esta organización facilita:
  - **Alta disponibilidad** con Keepalived.
  - **Gestión por SFTP** segura usando el usuario enjaulado `superwebmaster`.
  - Separación clara de contenidos por servidor.

---
