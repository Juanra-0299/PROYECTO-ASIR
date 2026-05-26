# Guía de Conexión Remota Segura con Tailscale y AnyDesk

Esta guía explica cómo configurar un acceso remoto seguro a un servidor o infraestructura de laboratorio **sin necesidad de abrir puertos en el router**, utilizando **Tailscale** para la conectividad VPN y **AnyDesk** para administración remota gráfica.

---

# Parte 1: Instalación y Configuración de Tailscale

## ¿Qué es Tailscale?

**Tailscale** es una VPN moderna basada en **WireGuard** que permite conectar dispositivos de forma segura a través de internet sin necesidad de abrir puertos ni configurar NAT en el router.

En este caso, se utilizará para acceder remotamente a:

- **Servidor Proxmox**
- **Router OPNsense**
- **Servidor WordPress**
- Otros servicios internos de la red

---

## Paso 1: Instalación de Tailscale

Instalamos Tailscale en el servidor Linux donde queremos habilitar el acceso remoto.

### Descargar e instalar Tailscale

```bash
wget https://tailscale.com/install.sh
sudo bash install.sh
```

Una vez finalizada la instalación, tendremos disponible el comando `tailscale`.

---

## Paso 2: Inicio de Sesión y Vinculación del Equipo

Ahora iniciaremos el servicio y vincularemos el equipo a nuestra cuenta de Tailscale.

### Activar Tailscale

```bash
sudo tailscale up
```

Al ejecutar el comando, aparecerá un enlace similar a este:

```text
https://login.tailscale.com/...
```

Copiamos el enlace y lo abrimos en el navegador.

### Iniciar sesión

Podemos iniciar sesión utilizando:

- Cuenta de GitHub
- Google
- Microsoft
- Correo electrónico

En este caso se utilizó **GitHub**.

Una vez autenticado, el dispositivo aparecerá automáticamente en el panel web de Tailscale.

---

## Paso 3: Habilitar SSH Seguro sobre Tailscale

Tailscale permite habilitar conexiones SSH seguras sin necesidad de exponer el puerto `22` a internet.

### Activar SSH

```bash
sudo tailscale set --ssh
```

Con esto el equipo ya aceptará conexiones SSH desde otros dispositivos pertenecientes a la misma red privada de Tailscale.

Podemos verificar el dispositivo desde el panel de administración web de Tailscale.

---

# Parte 2: Configuración del Cliente (Equipo de Casa)

Ahora instalaremos **Tailscale** también en el ordenador desde el que realizaremos la conexión remota.

Repetimos exactamente los mismos pasos:

```bash
wget https://tailscale.com/install.sh
sudo bash install.sh
```

Después iniciamos sesión:

```bash
sudo tailscale up
```

Una vez iniciado, el cliente podrá visualizar todos los dispositivos de nuestra red Tailscale.

---

## Obtener la IP de Tailscale del Servidor

En el cliente:

1. Hacemos clic derecho sobre el icono de **Tailscale**.
2. Entramos en:

```text
Network Devices > My Devices
```

3. Seleccionamos el equipo remoto.
4. Copiamos su dirección IP de Tailscale.

---

# Parte 3: Creación de un Túnel SSH

Una vez obtenida la IP, crearemos un túnel SSH para acceder a los servicios internos de la red del aula.

## ¿Qué hace un túnel SSH?

Un túnel SSH permite **redirigir puertos internos de una red remota hacia nuestro propio equipo local**, haciendo que servicios privados parezcan ejecutarse en nuestro ordenador.

---

## Crear el túnel SSH

Abrimos una terminal en el equipo cliente y ejecutamos:

```bash
ssh -L 8006:172.16.0.2:8006 \
-L 80:172.16.0.1:80 \
-L 8080:10.0.0.11:80 \
[usuario]@[ip_de_tailscale]
```

### Explicación del comando

La sintaxis es:

```text
-L [puerto_local]:[ip_destino]:[puerto_remoto]
```

En este caso:

| Servicio | IP remota | Puerto | Puerto Local |
|-----------|------------|---------|---------------|
| Proxmox | 172.16.0.2 | 8006 | 8006 |
| OPNsense | 172.16.0.1 | 80 | 80 |
| WordPress | 10.0.0.11 | 80 | 8080 |

### Sustituir valores

Debes cambiar:

```text
[usuario]
```

Por el usuario del servidor Linux.

Y:

```text
[ip_de_tailscale]
```

Por la IP copiada desde Tailscale.

Ejemplo:

```bash
ssh -L 8006:172.16.0.2:8006 \
-L 80:172.16.0.1:80 \
-L 8080:10.0.0.11:80 \
jesus@100.x.x.x
```

> **Importante:** La terminal debe permanecer abierta mientras quieras mantener el túnel activo.

---

# Parte 4: Acceso a los Servicios

Una vez activo el túnel SSH, podremos acceder a los servicios internos desde el navegador usando `localhost`.

## Acceso a Proxmox

```text
https://127.0.0.1:8006
```

> El navegador puede advertir sobre un certificado autofirmado. Es normal; basta con aceptar el riesgo y continuar.

---

## Acceso a WordPress

```text
http://127.0.0.1:8080
```

---

## Acceso a OPNsense

```text
http://127.0.0.1
```

---

# Parte 5: Alternativa Avanzada — Tailscale Subnet Routing

El túnel SSH funciona muy bien, pero obliga a:

- Ejecutar el comando manualmente
- Mantener la terminal abierta

Como alternativa, podemos utilizar **Subnet Routing**, permitiendo que el servidor actúe como un router para toda la red.

## Habilitar el reenvío de paquetes

Ejecutamos:

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf

echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
```

---

## Publicar las rutas de red

Ejecutamos:

```bash
sudo tailscale up --advertise-routes=172.16.0.0/24,10.0.0.0/24 --ssh
```

### ¿Qué significa esto?

Estamos anunciando las siguientes redes:

| Red | Función |
|------|----------|
| 172.16.0.0/24 | Infraestructura interna |
| 10.0.0.0/24 | Red de servidores |

---

## Aprobar las rutas

Ahora accedemos al panel web de Tailscale:

1. Entramos en **Admin Console**
2. Vamos a **Machines**
3. Seleccionamos el servidor
4. Entramos en:

```text
Edit Route Settings
```

5. Aprobamos las rutas anunciadas.

---

## Acceso directo sin túneles

A partir de este momento **ya no necesitaremos el túnel SSH**.

Podremos acceder directamente usando las IP reales.

### Proxmox

```text
https://172.16.0.2:8006
```

### WordPress

```text
http://10.0.0.11:8080
```

### OPNsense

```text
http://172.16.0.1
```

---

# Parte 6: Instalación de AnyDesk

## ¿Por qué usar AnyDesk?

AnyDesk permite administrar remotamente escritorios gráficos, siendo útil para:

- Supervisar el servidor
- Gestionar interfaces gráficas
- Soporte remoto

---

## Instalación de AnyDesk

Ejecutamos:

```bash
wget --max-redirect 1 --trust-server-names 'https://anydesk.com/en/downloads/thank-you?dv=deb_64' -O anydesk.deb

sudo apt install ./anydesk.deb
```

---

# Parte 7: Solución del Problema con Wayland

Al intentar conectarse, puede aparecer un error porque **AnyDesk no es compatible con Wayland**.

La solución consiste en instalar **Xfce** y utilizar **LightDM**.

## Instalar entorno gráfico compatible

```bash
sudo apt update && sudo apt install xfce4 xfce4-goodies lightdm -y
```

---

## Reconfigurar el gestor gráfico

Ejecutamos:

```bash
sudo dpkg-reconfigure gdm3
```

En el menú desplegable seleccionamos:

```text
lightdm
```

---

## Seleccionar sesión Xfce

En la pantalla de inicio de sesión:

1. Hacemos clic en el selector de sesión.
2. Elegimos:

```text
Xfce Session
```

3. Iniciamos sesión normalmente.

---

# Parte 8: Configuración de Seguridad en AnyDesk

Para evitar accesos no autorizados, configuramos un acceso mediante contraseña.

## Configurar contraseña de acceso

Abrimos:

```text
AnyDesk → Configuración → Acceso
```

Después:

1. **Desbloquear el control de seguridad**
2. **Cambiar contraseña de este puesto de trabajo**

En este caso se configuró:

```text
2Asir
```

> En un entorno real se recomienda utilizar contraseñas más seguras.

---

## Recomendaciones de Seguridad

También se recomienda:

- Restringir accesos no autorizados
- Excluir el equipo del descubrimiento automático
- Limitar usuarios permitidos

Esto mejora significativamente la seguridad del sistema remoto.

---

# Conclusión

Con esta configuración hemos conseguido:

✅ Acceso remoto seguro mediante **Tailscale**  
✅ Conexión SSH privada sin abrir puertos  
✅ Acceso a servicios internos mediante túneles SSH  
✅ Alternativa avanzada con **Subnet Routing**  
✅ Administración gráfica mediante **AnyDesk**
