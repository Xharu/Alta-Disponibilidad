# 🌐 Configuración Completa de DHCP + DNS (BIND9) para la Red Local

Este documento detalla el procedimiento paso a paso para configurar un servidor **DHCP + DNS (BIND9)** en Debian/Ubuntu, utilizado para la red de **alta disponibilidad con Keepalived y Apache2**.

- Interfaz de red: `ens33`  
- Subred: `192.168.100.0/24`  
- IP virtual del clúster: `192.168.100.100`  

---

## 📋 Requisitos Previos

* Tener acceso de superusuario (`sudo`).
* Servidor Debian/Ubuntu actualizado.
* Conectividad básica de red en `192.168.100.0/24`.
* IP del servidor DHCP/DNS: `192.168.100.10` (ejemplo).

---

## 🚀 Pasos de Configuración

1. **Instalación de Servicios**

    - Actualizamos repositorios:
        ```
        sudo apt update
        ```

    - Instalamos el servidor DHCP:
        ```
        sudo apt install isc-dhcp-server -y
        ```

    - Definimos la interfaz que atenderá DHCP en `/etc/default/isc-dhcp-server`:
        ```
        sudo nano /etc/default/isc-dhcp-server
        ```
        Contenido relevante:
        ```
        INTERFACESv4="ens33"
        INTERFACESv6=""
        ```

    - Instalamos el servidor DNS BIND9 y utilidades:
        ```
        sudo apt install bind9 bind9utils bind9-doc -y
        ```

    - Propósito:
        - `isc-dhcp-server`: asigna direcciones IP automáticamente.
        - `bind9`: resuelve nombres internos como `angel.usfx.edu`, `www.keepalived.usfx.edu`, etc.

2. **Configuración del Servidor DHCP**

    - Editamos el archivo principal:
        ```
        sudo nano /etc/dhcp/dhcpd.conf
        ```

    - Contenido de ejemplo:
        ```
        option domain-name "usfx.edu";
        option domain-name-servers 192.168.100.6;

        default-lease-time 86400;       # 24 horas
        max-lease-time 604800;          # 7 días

        authoritative;
        ddns-update-style none;

        subnet 192.168.100.0 netmask 255.255.255.0 {
            # Rango dinámico para clientes
            range 192.168.100.11 192.168.100.99;
            range 192.168.100.101 192.168.100.254;

            # Gateway y máscara de subred
            option routers 192.168.100.1;
            option broadcast-address 192.168.100.255;
            option subnet-mask 255.255.255.0;
        }
        ```

    - Explicación breve:
        - `option domain-name`: define el dominio local `usfx.edu`.
        - `option domain-name-servers`: indica el DNS que usarán los clientes (`192.168.100.10`).
        - `authoritative`: este servidor es la autoridad de la subred.
        - `subnet`: define red, rangos de IP y puerta de enlace.

3. **Configuración del Servidor DNS (BIND9)**

    3.1 **Definir la zona DNS**

    - Editamos el archivo de zonas locales:
        ```
        sudo nano /etc/bind/named.conf.local
        ```

    - Agregamos la zona:
        ```
        zone "usfx.edu" {
            type master;
            file "/etc/bind/db.usfx.edu";
        };
        ```

    - Propósito:
        - Indicar que el servidor es **master** para el dominio `usfx.bo`.
        - Definir el archivo donde estarán los registros DNS.

    3.2 **Crear el archivo de zona**

    - Creamos o editamos el archivo:
        ```
        sudo nano /etc/bind/db.usfx.edu
        ```

    - Contenido de ejemplo:
        ```
        $TTL    604800
        @       IN      SOA     ns1.usfx.edu. root.usfx.edu. (
                          301120252         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL

        ; Servidor de Nombres
        @       IN      NS      ns1.usfx.edu.
        @       IN      A       192.168.100.6

        ; Definición del Host ns1
        ns1     IN      A       192.168.100.6

        ; --- IP Virtual para alta disponibilidad ---
        www.keepalived   IN  A   192.168.100.100
        keepalived       IN  A   192.168.100.100

        ; Registros de servidores reales
        www.infraestructura  IN  A   192.168.100.2
        infraestructura      IN  A   192.168.100.2

        www.angel            IN  A   192.168.100.3
        angel                IN  A   192.168.100.3

        www.brayan           IN  A   192.168.100.4
        brayan               IN  A   192.168.100.4

        www.eli              IN  A   192.168.100.5
        eli                  IN  A   192.168.100.5
        ```

    - Explicación breve:
        - SOA/NS: definen el servidor de nombres principal (`ns1.usfx.edu`).
        - Registros `A`: asignan cada nombre a una IP.
        - `www.keepalived` y `keepalived` apuntan a la **IP virtual 192.168.100.100**, usada por Keepalived para alta disponibilidad.

4. **Reinicio y Verificación de Servicios**

    4.1 **Reiniciar servicios**

    - Reiniciamos DHCP y BIND9 para aplicar los cambios:
        ```
        sudo systemctl restart isc-dhcp-server
        sudo systemctl restart bind9
        ```

    4.2 **Verificar estado**

    - Comprobamos que ambos servicios estén activos:
        ```
        sudo systemctl status isc-dhcp-server
        sudo systemctl status bind9
        ```

5. **Pruebas de Funcionamiento**

    5.1 **Probar resolución DNS**

    - Desde el servidor o un cliente, probamos consultas:
        ```
        dig @192.168.100.6 angel.usfx.bo
        dig @192.168.100.6 www.keepalived.usfx.bo
        ```

    - Esperado:
        - `angel.usfx.bo` → `192.168.100.3`
        - `www.keepalived.usfx.bo` → `192.168.100.100`

    5.2 **Probar DHCP en un cliente**

    - Conectamos un cliente a la red `192.168.100.0/24`.
    - Verificamos la IP obtenida:
        ```
        ip a
        ```

    - Debe recibir una IP dentro de los rangos:
        - `192.168.100.11–99` o `192.168.100.101–254`
        - Gateway: `192.168.100.1`
        - DNS: `192.168.100.6`

---

## ✅ Observaciones Finales

- La interfaz de red usada por DHCP y DNS es `ens33`.
- DHCP asigna parámetros de red automáticamente (IP, gateway, DNS, dominio).
- BIND9 resuelve nombres internos `*.usfx.bo` y asigna la IP virtual `192.168.100.100` a `keepalived`.
- Gracias a la IP virtual:
  - Si el nodo MASTER cae, un BACKUP toma la IP.
  - Los clientes siguen usando el mismo nombre (`www.keepalived.usfx.bo`) sin notar la falla.
- La configuración es escalable: basta con agregar nuevos registros `A` en `db.usfx.bo` para incorporar más servidores.

---
