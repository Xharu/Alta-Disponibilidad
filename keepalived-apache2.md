# 🛡️ Configuración de Alta Disponibilidad con Keepalived + Apache2

Este documento detalla paso a paso la configuración de un clúster **tolerante a fallos (High Availability)** utilizando **Keepalived + Apache2**, implementando un sistema **MASTER – BACKUP** con una **IP virtual compartida**.[web:3][web:8]

---

## 📘 Información del Proyecto

**Proyecto:** ALTA DISPONIBILIDAD – TOLERANTE A FALLOS  
**Materia:** Infraestructura, Plataformas Tecnológicas y Redes  
**Docente:** Quispe Ortega Lucio Marcelo  
**Universitarios:**  
- Mauricio Torrejon Miguel Ángel  
- Elizabeth  
- Cruz Trujillo Brayann  

---

## 📋 Requisitos Previos

* Sistema basado en **Debian/Ubuntu**.[web:3]  
* Acceso como **sudo**.  
* Paquetes necesarios: `apache2`, `keepalived`.[web:5]  
* Interfaz de red activa (`ens37`).  
* **IP Virtual compartida:** `192.168.100.100`.  

---

## 🚀 Pasos de Configuración del Nodo PRINCIPAL (MASTER)

1. **Preparación del Sistema**

    - Actualizamos los repositorios e instalamos Apache2 + Keepalived:
        ```
        sudo apt update
        sudo apt install apache2 keepalived -y
        ```

    - Verificamos la interfaz de red:
        ```
        ip a
        ```

2. **Configuración de Keepalived (MASTER)**

    - Editamos el archivo de configuración principal:
        ```
        sudo nano /etc/keepalived/keepalived.conf
        ```

    - Contenido recomendado:
        ```
        vrrp_script check_apache {
            script "/etc/keepalived/check_apache.sh"
            interval 2
            weight -30
        }

        vrrp_instance VI_1 {
            state MASTER
            interface ens37
            virtual_router_id 51
            priority 100
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass secret
            }
            virtual_ipaddress {
                192.168.100.100
            }
            track_script {
                check_apache
            }
        }
        ```

    - Reiniciamos el servicio Keepalived:
        ```
        sudo systemctl restart keepalived
        ```

3. **Subida de Archivos al Servidor esta configuracion se realziara en los 4 servidores proxy, angel brayan y eli(ejemplo con SCP)**

    - Desde Windows hacia el servidor Linux:
        ```
        scp "E:\2-2025\SIS313\final\acreditacionCICO.zip" angel@192.168.100.12:/home/angel
        ```

4. **Configuración del Sitio Web en el MASTER**

    - Creamos la carpeta principal del sitio:
        ```
        sudo mkdir -p /var/www/html/master
        ```

    - Movemos el archivo subido a la carpeta del sitio:
        ```
        sudo mv /home/angel/acreditacionCICO.zip /var/www/html/master/
        cd /var/www/html/master
        ```

    - Instalamos `unzip` y descomprimimos el archivo:
        ```
        sudo apt install unzip -y
        sudo unzip acreditacionCICO.zip
        ```

    - Movemos el contenido y limpiamos archivos sobrantes:
        ```
        sudo mv acreditacionCICO/* .
        sudo rmdir acreditacionCICO
        sudo rm acreditacionCICO.zip
        ```

    - Asignamos permisos correctos a Apache:
        ```
        sudo chown -R www-data:www-data /var/www/html/master
        sudo chmod -R 755 /var/www/html/master
        ```

    - Creamos el VirtualHost:
        ```
        sudo nano /etc/apache2/sites-available/master.conf
        ```

    - Contenido del VirtualHost:
        ```
        <VirtualHost *:80>
            ServerName 192.168.100.100
            DocumentRoot /var/www/html/master

            <Directory /var/www/html/master>
                AllowOverride All
                Require all granted
            </Directory>
        </VirtualHost>
        ```

    - Activamos el sitio y recargamos Apache:
        ```
        sudo a2ensite master
        sudo a2dissite 000-default
        sudo systemctl reload apache2
        ```

5. **Script de Comprobación de Apache para Keepalived en todos los servidores web**

    - Creamos el script de chequeo:
        ```
        sudo nano /etc/keepalived/check_apache.sh
        ```

    - Contenido del script:
        ```
        #!/bin/bash
        systemctl status apache2 > /dev/null 2>&1
        if [ $? -eq 0 ]; then
            exit 0
        else
            exit 1
        fi
        ```

    - Damos permisos de ejecución:
        ```
        sudo chmod 755 /etc/keepalived/check_apache.sh
        ```

    - Habilitamos y reiniciamos Keepalived:
        ```
        sudo systemctl enable keepalived
        sudo systemctl restart keepalived
        ```

---

## 🔁 Configuración de los Nodos BACKUP

Todos los nodos BACKUP utilizan la misma lógica de configuración de **Keepalived**, cambiando solo la **prioridad** y, si se desea, el directorio del sitio web para identificar a cada nodo.[web:3][web:8]

---

## 🔵 BACKUP 1 – Angel (Prioridad 90)

1. **Configuración de Keepalived (BACKUP 1)**

    - Editamos `/etc/keepalived/keepalived.conf`:
        ```
        sudo nano /etc/keepalived/keepalived.conf
        ```

    - Contenido:
        ```
        vrrp_script check_apache {
            script "/etc/keepalived/check_apache.sh"
            interval 2
            weight -30
        }

        vrrp_instance VI_1 {
            state BACKUP
            interface ens37
            virtual_router_id 51
            priority 90
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass secret
            }
            virtual_ipaddress {
                192.168.100.100
            }
            track_script {
                check_apache
            }
        }
        ```

---

## 🟡 BACKUP 2 – Brayan (Prioridad 80)

1. **Configuración de Keepalived (BACKUP 2)**

    - Editamos `/etc/keepalived/keepalived.conf`:
        ```
        sudo nano /etc/keepalived/keepalived.conf
        ```

    - Contenido:
        ```
        vrrp_script check_apache {
            script "/etc/keepalived/check_apache.sh"
            interval 2
            weight -35
        }

        vrrp_instance VI_1 {
            state BACKUP
            interface ens37
            virtual_router_id 51
            priority 80
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass secret
            }
            virtual_ipaddress {
                192.168.100.100
            }
            track_script {
                check_apache
            }
        }
        ```

---

## 🟣 BACKUP 3 – Eli (Prioridad 70)

1. **Configuración de Keepalived (BACKUP 3)**

    - Editamos `/etc/keepalived/keepalived.conf`:
        ```
        sudo nano /etc/keepalived/keepalived.conf
        ```

    - Contenido:
        ```
        vrrp_script check_apache {
            script "/etc/keepalived/check_apache.sh"
            interval 2
            weight -30
        }

        vrrp_instance VI_1 {
            state BACKUP
            interface ens37
            virtual_router_id 51
            priority 70
            advert_int 1
            authentication {
                auth_type PASS
                auth_pass secret
            }
            virtual_ipaddress {
                192.168.100.100
            }
            track_script {
                check_apache
            }
        }
        ```

---

## 🧪 Comandos de Prueba del Servicio Apache

1. **Detener Apache (simular caída del servicio)**

    ```
    sudo systemctl stop apache2
    ```

2. **Iniciar Apache**

    ```
    sudo systemctl start apache2
    ```

3. **Reiniciar Apache**

    ```
    sudo systemctl restart apache2
    ```

4. **Ver estado de Apache**

    ```
    sudo systemctl status apache2
    ```

