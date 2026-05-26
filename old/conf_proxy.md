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
