# ⚙️ Usuario Restringido con Chroot para SFTP

Este documento detalla el procedimiento paso a paso para crear un usuario **restringido a SFTP** usando **Chroot** en un servidor Linux (Ubuntu/Debian), ideal para administrar archivos web de forma segura.

---

## 📋 Requisitos Previos

* Tener acceso de superusuario (`sudo`).
* Servidor con `openssh-server` instalado y activo.
* Directorio donde se alojarán los archivos web (en este ejemplo: `/srv/ftp/websites/rootwebservers`).

---

## 🎯 Objetivo de la Configuración

- Crear un usuario especial `superwebmaster` que:
  - Solo pueda usar **SFTP** (sin acceso a shell SSH).
  - Esté confinado a una **“jaula” Chroot**.
- Aumentar la seguridad evitando que el usuario navegue por todo el sistema.

---

## 🚀 Pasos de Configuración en los 4 servidores proxy, angel, brayann y eli

1. **Preparar la “jaula” (directorio Chroot):**

    - Creamos el directorio que actuará como raíz del entorno enjaulado:
        ```
        sudo mkdir -p /srv/ftp/websites/rootwebservers
        ```

    - Propósito:
        - Este será el directorio raíz que verá `superwebmaster`.
        - Todo su acceso quedará limitado a esta ruta.

2. **Crear el usuario restringido:**

    - Creamos el usuario con home dentro del chroot y sin shell interactiva:
        ```
        sudo useradd -d /srv/ftp/websites/rootwebservers -s /usr/sbin/nologin superwebmaster
        ```

    - Propósito:
        - `-d /srv/ftp/websites/rootwebservers`: define el home dentro de la jaula.
        - `-s /usr/sbin/nologin`: impide acceso a una shell SSH normal.

3. **Asignar contraseña al usuario:**

    - Definimos una contraseña para que pueda autenticarse por SFTP:
        ```
        sudo passwd superwebmaster
        ```

    - Propósito:
        - Permitir inicio de sesión SFTP con usuario/contraseña.
        - Sin contraseña, no podría conectarse.

4. **Configurar permisos del directorio raíz del Chroot:**

    - Establecemos propietario y permisos adecuados:
        ```
        sudo chown root:root /srv/ftp/websites/rootwebservers
        sudo chmod 755 /srv/ftp/websites/rootwebservers
        ```

    - Propósito:
        - La raíz del chroot **debe** pertenecer a `root` (requisito de OpenSSH).
        - Evita que el usuario pueda modificar la raíz y escapar de la jaula.

5. **Configurar SSH para usar Chroot + SFTP:**

    - Editamos el archivo de configuración de SSH:
        ```
        sudo nano /etc/ssh/sshd_config
        ```

    - Añadimos o ajustamos las siguientes líneas (normalmente al final del archivo):

        ```
        # Usar el SFTP interno de OpenSSH
        Subsystem sftp internal-sftp

        # Aplicar chroot solo al usuario superwebmaster
        Match User superwebmaster
            ChrootDirectory %h
            ForceCommand internal-sftp
            AllowTcpForwarding no
            X11Forwarding no
        ```

    - Propósito:
        - `Subsystem sftp internal-sftp`: usa el SFTP interno de OpenSSH.
        - `Match User superwebmaster`: aplica esta política solo a ese usuario.
        - `ChrootDirectory %h`: fija como raíz el home del usuario (`/srv/ftp/websites/rootwebservers`).
        - `ForceCommand internal-sftp`: fuerza el uso exclusivo de SFTP, sin shell.
        - `AllowTcpForwarding no` y `X11Forwarding no`: deshabilitan reenvíos para mayor seguridad.

6. **Reiniciar el servicio SSH:**

    - Aplicamos los cambios de configuración:
        ```
        sudo systemctl restart sshd
        ```

    - Propósito:
        - Recargar la configuración de SSH y activar el chroot para `superwebmaster`.

7. **Verificación de acceso SFTP:**

    - Desde otro equipo o desde el mismo servidor, probamos la conexión SFTP:
        ```
        sftp superwebmaster@<IP_DEL_SERVIDOR>
        ```

    - Propósito:
        - Comprobar que el usuario:
            - Puede autenticarse correctamente.
            - Solo ve el contenido dentro de `/srv/ftp/websites/rootwebservers`.
            - **No** tiene acceso a una shell (si intentas `ssh superwebmaster@<IP>` debería rechazar el acceso a shell).

---

## ✅ Resumen

Con esta configuración:

- `superwebmaster` es un usuario **exclusivo de SFTP**, sin shell.
- Está confinado a una **jaula Chroot** en `/srv/ftp/websites/rootwebservers`.
- Se reduce el riesgo de modificaciones accidentales o maliciosas fuera del entorno web autorizado.

---
  

