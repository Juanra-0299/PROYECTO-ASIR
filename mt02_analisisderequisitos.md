# Análisis de requisitos de infraestructura

## Situación actual
La infraestructura de red ya está parcialmente implementada.

Contamos con:
- Router del departamento.
- Patch panel.
- Switch.
- Switch configurable.
- Proxy del departamento (a sustituir por nuestro proxy).
- Switch por cada aula.
- Patch panel en algunas de las aulas (habrá que poner patch panel en aquellas que no esté).

## Recursos necesarios

### Router proxy de tres interfaces
Se requiere un router multiinterfaz con las siguientes funciones:

1. **Interfaz de entrada**: proveer conectividad a Internet.
2. **Interfaz DMZ**: conectar la red desmilitarizada.
3. **Interfaz de alumnado**: distribuir Internet a las aulas.

### Servidor en la DMZ
Un servidor que alojará los siguientes servicios, algunos de ellos mediante **contenedores Docker** para mayor eficiencia y escalabilidad:

- **DNS**: resolución de nombres y filtrado de contenido.
- **Web**: servicio web del centro.
- **FTP**: repositorio de archivos.
- **Utilidades varias**: juegos, temperaturas, etc.

### Equipos en la red de alumnado
Un servidor dedicado que proporciona:

- **FOG**: despliegue y restauración de equipos sin necesidad de pendrive.
- **RADIUS**: autenticación centralizada.

## Configuración crítica del proxy/router

### Redirección obligatoria de DNS
El router debe configurarse para redirigir **todo el tráfico DNS** (puerto 53 UDP/TCP) desde la red de alumnado hacia el servidor DNS de la DMZ, independientemente de la configuración manual que realicen los usuarios. Esto bloquea intentos de bypass aunque configuren DNS públicos como 8.8.8.8 o Cloudflare.

**Cómo implementarlo según el tipo de router**:
- **Router Linux**: Utilizar reglas de firewall y NAT
- **Router comercial**: Crear políticas NAT mediante la consola de administración, especificando:
  - Red de origen: alumnado
  - Puerto destino: 53
  - Acción: redirigir a la IP del DNS de la DMZ
  - Protocolos: UDP y TCP

### Bloqueo de DNS sobre HTTPS (DoH - DNS over HTTPS)
Algunos navegadores como Chrome utilizan la opción de **búsqueda inteligente por HTTPS**, que resuelve nombres de dominio sin pasar por el DNS configurado del sistema. Para evitar este bypass:

- **Bloquear servicios DoH**: Crear reglas de firewall que denieguen tráfico HTTPS hacia servidores DoH conocidos (Google (8.8.8.8:443, 8.8.4.4:443), Cloudflare (1.1.1.1:443), Quad9 (9.9.9.9:443), etc.)
- **Bloquear a nivel de dominio**: Si el proxy tiene capacidad de inspección, filtrar dominios que ofrecen DoH (dns.google, cloudflare-dns.com, etc.)
