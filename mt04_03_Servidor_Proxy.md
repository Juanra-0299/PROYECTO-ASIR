# 3. Implementación de servidor proxy: Guía de Configuración y Enrutamiento Avanzado con OPNsense
## Requisitos
Siguiendo la [estructura actual](/mt03_diseno_tecnico.png) necesitamos un equipo con un mínimo de 3 tarjetas de red para separar la red de los alumnos de los servidores.

Para la gestión, seguridad y el enrutamiento avanzado de nuestra red usaremos **OPNsense**, un potente firewall de código abierto basado en FreeBSD que destaca por su robustez y su interfaz intuitiva. 

Para garantizar un rendimiento óptimo, especialmente teniendo en cuenta la escalabilidad del proyecto y por si en el futuro habilitamos servicios que consumen muchos recursos (como la inspección profunda de paquetes con un IDS/IPS tipo Suricata o Zenarmor), hemos ajustado el hardware de la máquina virtual.

## 1. Especificaciones de Hardware (Máquina Virtual)

* **Memoria RAM (6 GiB):** OPNsense es un sistema muy eficiente y puede funcionar perfectamente con 2 GB para tareas básicas de enrutamiento. Sin embargo, asignarle 6 GiB nos da un margen de seguridad excelente para evitar cuellos de botella al analizar el tráfico de la red en tiempo real, almacenar grandes tablas de estado (*State Tables*) y manejar múltiples conexiones simultáneas sin que el sistema empiece a paginar en disco.
* **Procesadores (3 cores):** Una cantidad óptima para manejar altos anchos de banda sin saturar la CPU. Es muy recomendable que a nivel de hipervisor nos aseguremos de tener habilitada la instrucción **AES-NI**, ya que esto permite a OPNsense acelerar por hardware el cifrado y descifrado de paquetes, algo vital si decidimos levantar túneles VPN en el futuro.
* **Disco Duro (64 GB SSD):** Un disco de este tamaño y tecnología rápida es fundamental. Un firewall no solo enruta, sino que audita. Necesitamos espacio y velocidad de escritura/lectura para almacenar los extensos logs del sistema, generar reportes gráficos de tráfico y guardar volcados de red para su posterior análisis sin que la latencia de almacenamiento sea un impedimento.

---

## 2. Arquitectura Lógica de Red

El servidor contará con **3 tarjetas de red (interfaces)** perfectamente segmentadas para aislar el tráfico, reducir los dominios de colisión/broadcast y mantener la seguridad perimetral del entorno:

1. **Red del departamento (WAN) - `192.168.1.241/24`:** Actúa como nuestra puerta al exterior. Es la interfaz «sucia» o no confiable que nos provee de conexión a Internet a través de la red general del centro.
2. **Red de los alumnos (LAN) - `172.16.0.1/16`:** Una subred amplia dedicada exclusivamente a los equipos de los estudiantes. El uso de una máscara `/16` (65.534 hosts disponibles) nos permite un gran volumen de direcciones IP, ideal para un entorno de laboratorio donde se pueden desplegar decenas de máquinas virtuales para prácticas masivas sin riesgo de agotamiento de direccionamiento.
3. **DMZ (Zona Desmilitarizada) - `10.0.0.0/8`:** Es una red virtual aislada donde alojaremos los servidores expuestos (servidores web, DNS público, etc.). Al igual que la LAN, tiene una máscara muy amplia para escalar el laboratorio. Un detalle crucial de seguridad es que esta red no está vinculada a ningún puerto físico en el servidor, lo que añade una capa extra de protección para prevenir accesos no autorizados mediante hardware físico conectado en el aula.

> 📝 **Nota sobre el Hipervisor (Proxmox):** > La interfaz de puente `vmbr0` está vinculada al puerto físico `nic1` (interfaz `enp3s0`) y tiene configurada la IP `192.168.1.240`, utilizando el gateway `192.168.1.1`. Esta red pertenece formalmente al departamento y será la encargada de dar vida a nuestra WAN.

> 💡 **Tip de Administración de Sistemas:**
> Cuando un servidor tiene múltiples tarjetas de red físicas, a veces es confuso saber a qué puerto corresponde cada cable. Para identificar físicamente cada interfaz de red en el servidor (haciendo parpadear el LED de la tarjeta de red en el chasis físico), podemos conectarnos por SSH o consola y usar el comando: 
> `sudo ethtool --identify [nombre-de-la-interfaz]`

---

## 3. Instalación y Asignación de Interfaces

Realizamos la instalación base de OPNsense. Durante el asistente de consola inicial, asignamos las interfaces virtuales de OPNsense a las tarjetas de red que configuramos previamente en el hipervisor.

* **Credenciales por defecto:** Tras la instalación son el usuario `root` y la contraseña `2Asirrouter` (las cuales cambiaremos inmediatamente después del primer login por motivos de seguridad).

En resumen, la red que provee la conexión a Internet y que nos une al resto del instituto (la del departamento) se configura lógicamente como la interfaz **WAN**, mientras que los segmentos de Alumnos y DMZ se configuran como interfaces internas de tipo **LAN** u **OPT** (Interfaces Opcionales), protegiéndolas bajo el paraguas del firewall.

Una vez completada la asignación y con los servicios básicos corriendo, accedemos al panel de administración web desde un navegador cliente utilizando cualquiera de las IPs internas del firewall.

> ⚠️ **Nota Técnica (Certificado SSL):** El navegador mostrará una advertencia de seguridad indicando que la conexión no es privada debido al uso de un certificado SSL/TLS autofirmado. Este es un comportamiento estándar y esperado en dispositivos de red perimetral recién instalados; simplemente añadimos la excepción y aceptamos el riesgo para continuar hacia el *dashboard*.

---

## 4. Configuración de Reglas de Firewall y Alias

La verdadera potencia de OPNsense radica en su motor de filtrado de paquetes. A continuación, desglosamos las configuraciones lógicas de seguridad implementadas:

### Uso de Alias
El primer paso como administradores consistió en crear dos alias (`Puertos_web` y `Puertos_web2`).
El uso de Alias es una **«Mejor Práctica»** fundamental en OPNsense. Nos permite agrupar múltiples puertos (ej: `80`, `443`, `8080`) o direcciones IP bajo un único nombre descriptivo. Si en el futuro necesitamos abrir un nuevo puerto web, en lugar de modificar diez reglas de firewall distintas, simplemente lo añadimos al alias y todas las reglas se actualizarán dinámicamente. Esto evita saturar la tabla con decenas de reglas individuales y reduce drásticamente el riesgo de errores humanos.

### Aislamiento de la DMZ (Zero Trust)
En la sección `Firewall -> Rules -> DMZ` hemos configurado dos reglas de seguridad vitales siguiendo los principios de **Zero Trust (Confianza Cero)**:

1. **Bloqueo hacia la LAN:** Bloquea explícitamente todo el tráfico originado en la DMZ que tenga como destino la red de los alumnos (LAN). La lógica es la *contención de daños*: si un atacante compromete un servidor público de nuestra DMZ (el principal vector de ataque), esta regla evitará que pueda pivotar y usar ese servidor como puente para atacar los ordenadores vulnerables de los alumnos.
2. **Salida a Internet:** Permite a la DMZ salir hacia Internet (cualquier destino que no sea una red privada) para que los servidores puedan resolver nombres, actualizar su paquetería y sincronizar la hora.

> 🛑 **¡IMPORTANTE! Evaluación de reglas (First Match):**
> Las reglas del firewall se procesan de forma secuencial de arriba hacia abajo (la primera regla que coincide es la que se aplica). Es **absolutamente crucial** que la regla de bloqueo hacia la LAN esté colocada siempre *por encima* de la regla que permite salir a Internet. Si estuviesen invertidas, la regla de «Permitir todo» se activaría primero y el tráfico malicioso pasaría libremente hacia la LAN.

### Políticas para el segmento de Alumnos
En `Firewall -> Rules -> Alumnos` se han diseñado tres reglas estratégicas para asegurar la productividad y el control del aula:

1. **Bloqueo de DoH (Puerto 853):** Se deniega explícitamente el tráfico TCP/UDP saliente hacia el puerto `853` (DNS over TLS / DNS over HTTPS). Los navegadores modernos intentan usarlo por defecto para cifrar las peticiones DNS, lo que impide auditar qué webs visitan los usuarios. Al bloquear este puerto, obligamos al navegador a hacer peticiones DNS tradicionales en texto plano (puerto `53`), garantizando que el departamento pueda auditar el tráfico, capturar paquetes para prácticas o aplicar filtrado de contenidos.
2. **Acceso interno (DMZ):** Se permite la comunicación dirigida hacia la zona desmilitarizada para que los alumnos puedan consumir los servicios locales, realizar sus prácticas contra servidores web de pruebas y hacer pings de comprobación.
3. **Salida general:** Una regla final de tipo «Allow All» que autoriza el tráfico estándar de navegación y servicios generales hacia Internet.

---

## 5. Configuración de NAT

En el menú de `Firewall -> NAT -> Port Forwarding (DNAT)` se han implementado directivas esenciales para exponer servicios controlados al exterior y gobernar el flujo interno de red manipulando las cabeceras de los paquetes en tiempo real:

### Port Forwarding (DMZ)
* **Gestión de DNS:** Se redirige el puerto externo `5380` hacia la IP interna `10.0.0.10` (panel de control de nuestro DNS).
  * *(Nota de seguridad: En producción real, exponer paneles de administración directamente al exterior es una mala práctica y deben estar protegidos tras una VPN. En este laboratorio, se realiza por comodidad lectiva).*
* **Tráfico Web (WordPress):** Se redirige el tráfico entrante por los puertos alternativos `8080` y `8443` directamente hacia los puertos web estándar (`80` HTTP y `443` HTTPS) del servidor `10.0.0.11` (plataforma de WordPress de pruebas).

### Secuestro DNS (Transparent DNS Proxy)
La joya de la corona de nuestra configuración es una regla de NAT crítica de intercepción. Esta política intercepta por la fuerza todo el tráfico UDP/TCP dirigido a cualquier destino por el puerto `53` (DNS) generado dentro de la red de los alumnos, y lo redirige (secuestra) hacia nuestro servidor interno `10.0.0.10` que ejecuta **Technitium DNS**.

**¿Qué logramos con esto?** Si un alumno avanzado decide modificar manualmente los ajustes de su tarjeta de red para usar los servidores DNS de Google (`8.8.8.8`) o Cloudflare (`1.1.1.1`) para saltarse nuestros bloqueos, no le servirá de nada. OPNsense interceptará esa petición de forma totalmente transparente para el usuario, cambiará la IP de destino «al vuelo» y la enviará a nuestro propio servidor Technitium. Así, mantenemos el control dictatorial y absoluto sobre las resoluciones de nombres en toda el aula.
