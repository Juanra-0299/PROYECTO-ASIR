# 4. Conexión remota
# Manual de Conexión Remota Segura con Tailscale y AnyDesk

Este documento detalla los pasos para establecer una **conexión remota a través de Internet de forma segura** hacia nuestro servidor o equipo de clase, **sin necesidad de abrir puertos en el router**.

Utilizaremos:

- **Tailscale** → para crear una red privada virtual (VPN).
- **AnyDesk** → como herramienta de soporte gráfico remoto.

---

# 🛠️ Parte 1: Instalación y Configuración de Tailscale

Tailscale nos permite crear una **red privada (VPN)** de forma rápida y sencilla.

## 1. Instalación en el Servidor (Linux)

Para instalar Tailscale en nuestro equipo Linux, ejecutamos los siguientes comandos:

```bash
wget https://tailscale.com/install.sh
sudo bash install.sh
📷 Análisis visual

En las imágenes capturadas de la terminal se observa cómo:

El sistema descarga el script install.sh mediante wget.
Posteriormente se desempaquetan e instalan los paquetes .deb necesarios para la arquitectura amd64.
Finalmente, la instalación se completa correctamente.
2. Autenticación y Habilitación de SSH

Una vez instalado, iniciamos el servicio:

sudo tailscale up
🔐 Inicio de sesión

Este comando generará una URL en la consola.

Debemos:

Copiar el enlace.
Abrirlo en el navegador web.
Autenticarnos con una cuenta.

En este caso práctico se utilizó una cuenta de GitHub para iniciar sesión.

🔓 Habilitar SSH en Tailscale

Para permitir conexiones seguras por SSH directamente a través de la red de Tailscale, ejecutamos:

sudo tailscale set --ssh
📷 Análisis visual

En la consola de administración web de Tailscale (Admin Console) se puede comprobar que el equipo aparece conectado y con el indicador SSH habilitado en color verde.

🚇 Parte 2: Acceso mediante Túneles SSH (Método 1)

Este método es muy seguro, aunque requiere mantener una terminal abierta durante toda la sesión.

1. Preparación en el equipo Cliente (Casa)

Instalamos Tailscale también en nuestro equipo personal.

Después:

Hacemos clic derecho sobre el icono de Tailscale en la barra de tareas de Windows.
Accedemos a:
Network devices > My Devices
Seleccionamos nuestro equipo de clase.

Esto copiará automáticamente la IP de Tailscale al portapapeles.

📷 Análisis visual

Las capturas muestran el menú desplegable de Tailscale en Windows, donde aparecen los dispositivos de la red y se selecciona el equipo remoto para copiar su dirección IP.

2. Creación del Túnel SSH

Abrimos una terminal en nuestro equipo local y ejecutamos el siguiente comando:

ssh -L 8006:172.16.0.2:8006 \
-L 80:172.16.0.1:80 \
-L 8080:10.0.0.11:80 \
[usuario]@[ip_de_tailscale]

Nota: Debes sustituir [usuario] y [ip_de_tailscale] por los datos reales de tu servidor.

❓ ¿Qué hace este comando?

Este comando redirecciona los puertos de los servicios internos de la red de clase hacia nuestro propio localhost (127.0.0.1).

Ejemplo de servicios
Servicio	IP destino	Puerto
Proxmox	172.16.0.2	8006
OPNsense	172.16.0.1	80
WordPress	10.0.0.11	80

La sintaxis utilizada es:

-L [puerto_localhost]:[ip_destino]:[puerto_servicio]
📷 Análisis visual

Las imágenes muestran cómo:

Se ejecuta el comando SSH.
Se acepta la huella de seguridad (fingerprint).
Se establece correctamente el túnel.

Posteriormente, desde el navegador web se accede a los servicios usando:

Accesos disponibles
Proxmox
https://127.0.0.1:8006

Será necesario aceptar el riesgo del certificado autofirmado.

WordPress
http://127.0.0.1:8080
OPNsense
http://127.0.0.1

⚠️ Importante: Debes mantener esta ventana de terminal abierta.
Si la cierras, el túnel dejará de funcionar.

🛣️ Parte 3: Acceso mediante Subnet Routing (Método 2 - Recomendado)

Para evitar abrir un túnel SSH cada vez que queremos trabajar, Tailscale permite que nuestro servidor actúe como router hacia toda la subred interna de clase.

1. Configuración del Enrutamiento en el Servidor

Primero habilitamos el reenvío de paquetes (IP Forwarding):

echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf

echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf

A continuación, anunciamos las rutas de nuestras subredes:

sudo tailscale up --advertise-routes=172.16.0.0/24,10.0.0.0/24 --ssh
2. Aprobación de las Rutas

Por motivos de seguridad, las rutas deben aprobarse manualmente desde la administración de Tailscale.

Pasos
Entrar al panel de administración web de Tailscale.
Ir a la pestaña:
Machines
Buscar nuestro servidor.
Seleccionar:
Edit route settings
Aprobar las rutas anunciadas.
📷 Análisis visual

Las capturas muestran:

La ejecución correcta de los comandos de red (sysctl).
El anuncio de rutas.
La ventana donde el administrador habilita:
10.0.0.0/24
172.16.0.0/24
3. Acceso Directo

Una vez configurado, ya no será necesario usar 127.0.0.1 ni crear túneles SSH.

Simplemente debemos tener Tailscale encendido en el equipo cliente.

Servicios disponibles
Proxmox
https://172.16.0.2:8006
WordPress
http://10.0.0.11:8080
OPNsense
http://172.16.0.1
🖥️ ANEXO: Configuración de AnyDesk para Monitorización

Se utiliza AnyDesk para la monitorización gráfica del servidor FOG.

Instalación de AnyDesk

Descargamos e instalamos el paquete .deb:

wget --max-redirect 1 --trust-server-names \
'https://anydesk.com/en/downloads/thank-you?dv=deb_64' \
-O anydesk.deb && sudo apt install ./anydesk.deb
❌ Problema con Wayland

Al intentar conectar por primera vez puede aparecer el siguiente error:

No admitido:
No se admite el servidor remoto de pantalla
(p. ej. Wayland)
¿Por qué ocurre?

Wayland bloquea la captura de pantalla necesaria para herramientas clásicas de control remoto como AnyDesk.

✅ Solución: Instalar Xfce + LightDM

Instalamos un entorno compatible:

sudo apt update && sudo apt install xfce4 xfce4-goodies lightdm -y && sudo dpkg-reconfigure gdm3

Durante la instalación debemos seleccionar:

lightdm
📷 Análisis visual

Tras reiniciar el sistema:

En la pantalla de inicio de sesión (Login).
Hacemos clic en el selector de entorno (esquina superior derecha).
Seleccionamos:
Sesión de Xfce

En lugar del entorno predeterminado basado en Wayland.

🔐 Configuración de Seguridad en AnyDesk
1. Configurar Contraseña

Ir a:

Configuración → Acceso →  
Desbloquear el control de seguridad →  
Cambiar la contraseña de este puesto de trabajo

En este caso se configuró:

2Asir
2. Restricción de Acceso

Se configuró una lista blanca, permitiendo el acceso únicamente a IDs específicos de AnyDesk.

Ruta:

Restricción de acceso

Además:

Se excluyó el dispositivo de la red de descubrimiento público.
Se limitaron accesos no autorizados.
✅ Conclusión

Gracias a esta configuración:

Podemos acceder remotamente sin abrir puertos en el router.
La conexión se realiza mediante una VPN privada segura (Tailscale).
Podemos acceder a los servicios internos de red de forma sencilla.
Disponemos de acceso gráfico gracias a AnyDesk.

El método recomendado es Subnet Routing, ya que evita tener que crear túneles SSH manualmente cada vez que queremos conectarnos.
