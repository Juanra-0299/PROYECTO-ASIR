# 1. Introducción y objetivos

## 1.1. Introducción y justificación
Este Proyecto Intermodular surge de la necesidad de actualizar la infraestructura de red de nuestro centro educativo debido a la obsolescencia del sistema de proxy actual y la demanda de una arquitectura más segura y eficiente.

Aprovechando la sustitución del dispositivo de borde (proxy), se rediseñará la topología lógica para transicionar de una red plana a una estructura segmentada y jerárquica. La pieza central de esta actualización es la implementación de una **Zona Desmilitarizada (DMZ)**, diseñada para alojar servicios accesibles públicamente y aislar el tráfico externo de la red interna (LAN) educativa y administrativa.

El proyecto simula un entorno de producción real para un instituto de Formación Profesional, integrando enrutamiento avanzado, segmentación mediante VLANs y servicios tan importantes como DNS, autenticación RADIUS centralizada y sistemas de despliegue automatizado (FOG Project).

## 1.2. Objetivos del proyecto

### Objetivo General
Diseñar, implementar y asegurar una infraestructura de red escalable que sustituya el sistema proxy actual, integrando seguridad perimetral mediante una DMZ y optimizando la gestión de servicios para los ciclos formativos (FPB, SMR y ASIR).

### Objetivos Específicos

**1. Reestructuración de la Topología de Red**
* Segmentar la red en tres zonas lógicas diferenciadas para garantizar la seguridad y el rendimiento:
    * **Zona WAN/Departamental:** Subred `192.168.1.0/24` (Conexión a Internet).
    * **Zona DMZ:** Subred `10.0.0.0/8` (Servidores públicos).
    * **Zona LAN/Alumnado:** Subred `172.16.0.0/16` (Red interna de aulas).
* Configurar el enrutamiento inter-VLAN para gestionar el tráfico entre los distintos grupos de trabajo (1SMR, 2SMR, 1ASIR, 2ASIR, etc).

**2. Implementación de Servicios en la DMZ**
* **DNS:** Configuración de resolución de nombres y filtrado de contenido web, optimizando el ancho de banda mediante caché.
* **Web:** Alojamiento del sitio web institucional del centro.
* **FTP (Opcional):** Despliegue de un repositorio de archivos accesible tanto interna como externamente.
* **Correo (Opcional):** Valoración de la implementación de un servidor de correo propio o su integración con soluciones externas.

**3. Optimización de la Red Interna (Servicios de Gestión)**
* **FOG Project:** Implementación en la red de distribución para la clonación y despliegue de imágenes de sistemas operativos vía red (PXE/Multicast), eliminando el uso de soportes físicos.
* **RADIUS:** Configuración de un servidor de autenticación para centralizar el control de acceso. Se integrará con el servicio FTP de la DMZ para evitar la gestión de usuarios locales, permitiendo el uso de credenciales corporativas (LDAP/AD).

**4. Conectividad Externa**
* **DDNS:** Configuración de DNS Dinámico en el router de borde para mitigar la rotación de IP pública dinámica y garantizar el acceso remoto a los servicios.

## 1.3. Alcance del proyecto
El proyecto abarca:

**-Ciclo de vida completo de la solución a nivel lógico y de software:**
1.  **Diseño:** Modelado de la topología en **Cisco Packet Tracer**, definiendo direccionamiento IP, VLANs y enrutamiento estático/dinámico.
2.  **Implementación:** Instalación y configuración de los servicios (DNS, Web, FTP, RADIUS, FOG) sobre sistemas operativos servidor (Linux/Windows Server).
3.  **Seguridad:** Definición de reglas de firewall y ACLs para controlar el tráfico entre la red Alumnos, la DMZ e Internet.
4.  **Validación:** Ejecución de un plan de pruebas de conectividad y funcionalidad de servicios.

**-Análisis, Mantenimiento y configuracion del hardware:**

* **Auditoría de Inventario y Compatibilidad:**
    * Evaluación de los recursos de hardware existentes (CPU, RAM, Almacenamiento) en los servidores destinados a virtualización (Proxmox/ESXi) para asegurar que soportan la carga de los nuevos servicios (RADIUS, FOG, Web, etc.).
    * Verificación de capacidades en equipos de interconexión: Soporte para estándares en switches y capacidad de procesamiento/throughput en el router para gestionar el tráfico de la DMZ sin cuellos de botella.

* **Mantenimiento Preventivo y Actualización:**
    * Actualización de **Firmware/IOS** en switches y routers para mitigar vulnerabilidades conocidas y asegurar compatibilidad con nuevas funcionalidades.
    * Configuración de niveles de **RAID** (software o hardware) en los servidores para garantizar la redundancia y disponibilidad de datos 

* **Configuración de Dispositivos Activos:**
    * Configuración de interfaces, enrutamiento estático/dinámico y servicios base (DHCP, NTP) en el hardware de red.
    * Segmentación de puertos en switches (Access/Trunk) acorde al diseño de VLANs propuesto.

* **Infraestructura Física Pasiva:**
    * Instalación/revisión de cableado estructurado (UTP/Fibra), canaletas, rosetas, paneles de parcheo (patch panels). Se utilizará el cableado ya certificado del edificio.
    * Incluye el montaje físico de armarios rack y la instalación eléctrica asociada (SAI/UPS)
