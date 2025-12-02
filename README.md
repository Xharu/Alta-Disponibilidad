# 📌 Proyecto: **ALTA DISPONIBILIDAD – TOLERANTE A FALLAS y Automatizado**

### Materia: **Infraestructura, Plataformas Tecnológicas y Redes**  
### Docente: **Quispe Ortega Lucio Marcelo**  
### Integrantes:
- **Mauricio Torrejon Miguel Ángel**  
- **Chungara Choque Elizabeth**  
- **Cruz Trujillo Brayann**

---

## 🖥️ Descripción del Proyecto

El proyecto **Alta Disponibilidad y Tolerancia a Fallos** tiene como objetivo implementar una arquitectura capaz de garantizar la continuidad del servicio incluso ante fallos inesperados.  
Se busca asegurar que los sistemas permanezcan operativos, minimizando los tiempos de inactividad mediante tecnologías de redundancia, balanceo de carga y recuperación automática.

---

## 🎯 Objetivos

- Diseñar una infraestructura que mantenga los servicios activos ante fallos de hardware o software.  
- Implementar técnicas de **redundancia**, **replicación** y **automatización**.  
- Evaluar el rendimiento y comportamiento del sistema bajo diferentes escenarios de falla.  
- Documentar la arquitectura y la solución final implementada.

---

## 🛠️ Tecnologías utilizadas

- Servidores Linux / Windows  
- Balanceadores de carga  
- Máquinas virtuales  
- Redes y protocolos de comunicación  
- **Apache2**  
- **Keepalived**  
- Scripts automatizados  
- **Rsync**  
- **Crontab**  
- **Bind9** (DNS)  
- **ISC DHCP Server**  
- **SFTP**

---

## 🧩 Arquitectura del Sistema

La arquitectura implementada está compuesta por **6 servidores**, donde el servidor principal es el **Proxy**, el cual posee la prioridad más alta.  
Este actúa como nodo central; en caso de que falle, los servidores sucesores —**Angel, Brayan y Eli**— entran en funcionamiento automáticamente gracias a **Keepalived**, manteniendo la página activa.

Todos los servicios se respaldan mediante *backups automáticos* programados a las **08:00, 12:00 y 22:00**.

Además:

- Se cuenta con un servidor **DHCP–DNS** que gestiona la asignación de direcciones y resolución de nombres.  
- Se utiliza **SFTP** para la modificación de archivos del sitio web, garantizando seguridad y evitando usos no autorizados.  

### 📸 Imagen de la arquitectura

<img width="1409" height="608" alt="image" src="https://github.com/user-attachments/assets/ddab418e-0173-4edb-abf3-71f45ff9706c" />

---

## 📄 Conclusiones

El proyecto demostró la importancia de contar con infraestructuras diseñadas para resistir fallos y mantener la continuidad del servicio.  
La alta disponibilidad es esencial en sistemas críticos, permitiendo reducir los tiempos de inactividad y mejorar la confiabilidad general del sistema.
