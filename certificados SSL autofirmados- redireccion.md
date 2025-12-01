# 🔒 Configuración Completa de HTTPS en Apache – Servidores Proxy, Angel, Brayan y Eli

El procedimiento para habilitar **HTTPS con certificados SSL autofirmados** y configurar la **redirección automática de HTTP → HTTPS** en los servidores `proxy`, `angel`, `brayan` y `eli`.

---

## 📋 Requisitos Previos Generales

* Sistema Linux Ubuntu.
* Apache2 instalado y funcionando.
* Acceso con permisos `sudo`.
* Sitios web alojados en:
  * Proxy/Infraestructura: `/srv/ftp/websites/rootwebservers/superwebmaster/infraestructura/public_html`
  * Angel: `/srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html`
  * Brayan: `/srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html`
  * Eli: `/srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html`

---

## 🌐 HTTPS – Servidor Proxy (Infraestructura)

1. **Activación del módulo SSL**

    - Habilitamos el módulo SSL y reiniciamos Apache:
        ```
        sudo a2enmod ssl
        sudo systemctl restart apache2
        ```

    - Explicación:
        - `a2enmod ssl`: activa el módulo SSL en Apache.
        - `systemctl restart apache2`: aplica los cambios.

2. **Crear el certificado SSL autofirmado**

    - Generamos un certificado y una clave privada:
        ```
        sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
          -keyout /etc/ssl/private/keepalived.key \
          -out /etc/ssl/certs/keepalived.crt
        ```

    - Explicación:
        - Clave RSA de 2048 bits.
        - Validez de 365 días.
        - Clave: `/etc/ssl/private/keepalived.key`
        - Certificado: `/etc/ssl/certs/keepalived.crt`

3. **Crear el VirtualHost HTTPS (puerto 443)**

    - Creamos el archivo de configuración:
        ```
        sudo nano /etc/apache2/sites-available/infraestructura-ssl.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:443>
            ServerName www.infraestructura.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/infraestructura/public_html

            <Directory /srv/ftp/websites/rootwebservers/superwebmaster/infraestructura/public_html>
                Options FollowSymLinks
                AllowOverride All
                Require all granted
                DirectoryIndex index.html
            </Directory>

            SSLEngine on
            SSLCertificateFile /etc/ssl/certs/keepalived.crt
            SSLCertificateKeyFile /etc/ssl/private/keepalived.key

            ErrorLog ${APACHE_LOG_DIR}/infraestructura-ssl-error.log
            CustomLog ${APACHE_LOG_DIR}/infraestructura-ssl-access.log combined
        </VirtualHost>
        ```

4. **Habilitar el sitio HTTPS**

    - Habilitamos el VirtualHost y recargamos Apache:
        ```
        sudo a2ensite infraestructura-ssl.conf
        sudo apache2ctl configtest
        sudo systemctl reload apache2
        ```

    - Explicación:
        - `a2ensite`: habilita el archivo de sitio.
        - `configtest`: verifica la sintaxis.
        - `reload`: aplica cambios sin cortar conexiones.

5. **Redirección HTTP → HTTPS**

    - Editamos el sitio HTTP:
        ```
        sudo nano /etc/apache2/sites-available/infraestructura.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:80>
            ServerName www.infraestructura.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            Redirect / https://www.infraestructura.usfx.edu/
            Redirect / https://www.keepalived.usfx.edu/

            DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/infraestructura/public_html

            <Directory /srv/ftp/websites/rootwebservers/superwebmaster/infraestructura/public_html>
                Options FollowSymLinks
                AllowOverride All
                Require all granted
                DirectoryIndex index.html
            </Directory>

            ErrorLog ${APACHE_LOG_DIR}/infraestructura-error.log
            CustomLog ${APACHE_LOG_DIR}/infraestructura-access.log combined
        </VirtualHost>
        ```

6. **Activar la redirección HTTP**

    - Habilitamos el sitio HTTP y recargamos Apache:
        ```
        sudo a2ensite infraestructura.conf
        sudo apache2ctl configtest
        sudo systemctl reload apache2
        ```

---

## ⭐ Resultado – Servidor Proxy

- HTTPS funcionando.
- Certificado autofirmado instalado.
- Redirección de HTTP → HTTPS activa.
- Apache sirviendo contenido seguro desde `infraestructura`.

---

## 📘 HTTPS – Servidor Angel

### 📋 Requisitos Específicos

* Dominio: `www.angel.usfx.edu`
* Carpeta web: `/srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html`

---

1. **Activar módulo SSL**

    ```
    sudo a2enmod ssl
    sudo systemctl restart apache2
    ```

2. **Crear certificado autofirmado**

    ```
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/ssl/private/angel.key \
      -out /etc/ssl/certs/angel.crt
    ```

3. **Crear VirtualHost HTTPS (443)**

    - Archivo:
        ```
        sudo nano /etc/apache2/sites-available/angel-ssl.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:443>
            ServerName www.angel.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html

            <Directory /srv/ftp/websites/rootwebservers/superwebmaster/angel/public_html>
                Options FollowSymLinks
                AllowOverride All
                Require all granted
                DirectoryIndex index.html
            </Directory>

            SSLEngine on
            SSLCertificateFile /etc/ssl/certs/angel.crt
            SSLCertificateKeyFile /etc/ssl/private/angel.key
        </VirtualHost>
        ```

4. **Habilitar HTTPS**

    ```
    sudo a2ensite angel-ssl.conf
    sudo apache2ctl configtest
    sudo systemctl reload apache2
    ```

5. **Redirección HTTP → HTTPS**

    - Archivo:
        ```
        sudo nano /etc/apache2/sites-available/angel.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:80>
            ServerName www.angel.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            Redirect / https://www.angel.usfx.edu/
            Redirect / https://www.keepalived.usfx.edu/

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

6. **Activar sitio HTTP con redirección**

    ```
    sudo a2ensite angel.conf
    sudo apache2ctl configtest
    sudo systemctl reload apache2
    ```

---

## ⭐ Resultado – Servidor Angel

- HTTPS activo.
- Certificado autofirmado funcionando.
- Redirección automática de HTTP → HTTPS habilitada.

---

## 📘 HTTPS – Servidor Brayan

### 📋 Requisitos Específicos

* Dominio: `www.brayan.usfx.edu`
* Carpeta web: `/srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html`

---

1. **Activar SSL**

    ```
    sudo a2enmod ssl
    sudo systemctl restart apache2
    ```

2. **Crear certificado autofirmado**

    ```
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/ssl/private/brayan.key \
      -out /etc/ssl/certs/brayan.crt
    ```

3. **Crear VirtualHost HTTPS**

    - Archivo:
        ```
        sudo nano /etc/apache2/sites-available/brayan-ssl.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:443>
            ServerName www.brayan.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html

            <Directory /srv/ftp/websites/rootwebservers/superwebmaster/brayan/public_html>
                Options FollowSymLinks
                AllowOverride All
                Require all granted
                DirectoryIndex index.html
            </Directory>

            SSLEngine on
            SSLCertificateFile /etc/ssl/certs/brayan.crt
            SSLCertificateKeyFile /etc/ssl/private/brayan.key
        </VirtualHost>
        ```

4. **Activar HTTPS**

    ```
    sudo a2ensite brayan-ssl.conf
    sudo apache2ctl configtest
    sudo systemctl reload apache2
    ```

5. **Redirección HTTP → HTTPS**

    - Archivo:
        ```
        sudo nano /etc/apache2/sites-available/brayan.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:80>
            ServerName www.brayan.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            Redirect / https://www.brayan.usfx.edu/
            Redirect / https://www.keepalived.usfx.edu/

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

6. **Activar sitio con redirección**

    ```
    sudo a2ensite brayan.conf
    sudo apache2ctl configtest
    sudo systemctl reload apache2
    ```

---

## ⭐ Resultado – Servidor Brayan

- HTTPS funcionando.
- Redirección HTTP → HTTPS activa.
- Certificado autofirmado correctamente instalado.

---

## 📘 HTTPS – Servidor Eli

### 📋 Requisitos Específicos

* Dominio: `www.eli.usfx.edu`
* Carpeta web: `/srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html`

---

1. **Activar módulo SSL**

    ```
    sudo a2enmod ssl
    sudo systemctl restart apache2
    ```

2. **Crear certificado SSL autofirmado**

    ```
    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/ssl/private/eli.key \
      -out /etc/ssl/certs/eli.crt
    ```

3. **Crear VirtualHost HTTPS**

    - Archivo:
        ```
        sudo nano /etc/apache2/sites-available/eli-ssl.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:443>
            ServerName www.eli.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            DocumentRoot /srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html

            <Directory /srv/ftp/websites/rootwebservers/superwebmaster/eli/public_html>
                Options FollowSymLinks
                AllowOverride All
                Require all granted
                DirectoryIndex index.html
            </Directory>

            SSLEngine on
            SSLCertificateFile /etc/ssl/certs/eli.crt
            SSLCertificateKeyFile /etc/ssl/private/eli.key
        </VirtualHost>
        ```

4. **Activar sitio HTTPS**

    ```
    sudo a2ensite eli-ssl.conf
    sudo apache2ctl configtest
    sudo systemctl reload apache2
    ```

5. **Crear redirección HTTP → HTTPS**

    - Archivo:
        ```
        sudo nano /etc/apache2/sites-available/eli.conf
        ```

    - Contenido:
        ```
        <VirtualHost *:80>
            ServerName www.eli.usfx.edu
            ServerAlias www.keepalived.usfx.edu

            Redirect / https://www.eli.usfx.edu/
            Redirect / https://www.keepalived.usfx.edu/

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

6. **Activar sitio con redirección**

    ```
    sudo a2ensite eli.conf
    sudo apache2ctl configtest
    sudo systemctl reload apache2
    ```

---

## ⭐ Resultado – Servidor Eli

- HTTPS activo.
- Certificado autofirmado funcionando.
- Redirección HTTP → HTTPS correctamente configurada.

---
