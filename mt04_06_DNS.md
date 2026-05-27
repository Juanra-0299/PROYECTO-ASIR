# Guía de Despliegue y Configuración de Technitium DNS Server

**Technitium DNS Server** es un software de servidor de Sistema de Nombres de Dominio (DNS) gratuito, de código abierto y multiplataforma[cite: 3]. Su implementación nos permite centralizar la resolución de nombres en nuestra red local, mejorar la privacidad y aplicar políticas estrictas de filtrado de contenido (bloqueo de publicidad, malware y sitios inapropiados) de forma totalmente transparente para el usuario final[cite: 3].

---

## 1. Despliegue y Recursos de Hardware

El servicio se ha desplegado sobre un **contenedor LXC (Linux Container)** con Ubuntu dentro de nuestro hipervisor Proxmox, aprovechando la ligereza y el mínimo overhead de la virtualización a nivel de sistema operativo[cite: 3]. 

Los recursos asignados para garantizar un servicio ágil y con capacidad de respuesta inmediata son los siguientes[cite: 3]:

| Recurso | Configuración |
| :--- | :--- |
| **Memoria RAM** | 2 GiB |
| **Procesador (CPU)** | 2 Cores |
| **Disco Duro** | 50 GB |

### Automatización de la Instalación
El proceso de despliegue se ha simplificado y automatizado ejecutando el script oficial del desarrollador mediante el siguiente comando en la consola del contenedor[cite: 3]:

```bash
curl -sSL [https://download.technitium.com/dns/install.sh](https://download.technitium.com/dns/install.sh) | sudo bash
```

* **Acceso a la Interfaz Web:** Una vez finalizada la instalación, el panel de gestión web queda expuesto en el puerto `5380`[cite: 3].
* **Credenciales de Administración:** Protegido inicialmente bajo el usuario `admin` y la contraseña `2Asirdns` (las cuales deben ser actualizadas tras el primer acceso por normativas de seguridad del sistema)[cite: 3].

---

## 2. Configuración y Filtrado de Contenido

La principal línea de defensa perimetral de este servidor es su motor de bloqueo de peticiones basado en reputación de dominios[cite: 3]. Para securizar el entorno educativo, se configuró el filtrado de la siguiente manera[cite: 3]:

1. Navegar en el panel a: `Settings` -> `Blocking` -> `Allow / Block List URLs`[cite: 3].
2. Utilizar la opción **Quick Add** para añadir listas de bloqueo masivas[cite: 3].
3. Se ha implementado la **lista consolidada de Steven Black** (versión extendida que incluye: *adware + malware + porn + social*)[cite: 3]. Esto garantiza un entorno escolar seguro, mitigando vectores de infección por malware y restringiendo el acceso a material inapropiado o redes sociales de distracción durante las horas de clase[cite: 3].

### Integración con OPNsense (DNS Sniffing)
Este esquema de filtrado es infalible gracias a la regla de **Secuestro DNS (Transparent DNS Proxy)** configurada previamente en el firewall OPNsense[cite: 3]. Cualquier consulta dirigida al puerto `53` exterior por parte de un cliente es interceptada "al vuelo" y redirigida obligatoriamente hacia la IP de este contenedor (`10.0.0.10`)[cite: 3]. De esta forma, si un usuario intenta configurar manualmente DNS públicos (como `8.8.8.8` o `1.1.1.1`), su tráfico es capturado y filtrado de igual manera por Technitium[cite: 3].

---

## 3. Resolución de Nombres Internos (Zonas Locales)

Además de actuar como filtro perimetral, Technitium se comporta como el servidor DNS autoritativo para nuestra infraestructura local, centralizando el acceso a los servidores internos bajo el dominio de laboratorio **`iespoveda.com`**[cite: 3].

Esto elimina la necesidad de recordar direcciones IP complejas, permitiendo el acceso mediante nombres de dominio completamente cualificados (**FQDN**)[cite: 3]:

| Nombre FQDN | Dirección IP | Propósito / Servicio Asociado |
| :--- | :--- | :--- |
| `dns.iespoveda.com` | `10.0.0.10` | Interfaz de gestión de este servidor Technitium[cite: 3] |
| `web.iespoveda.com` | `10.0.0.11` | Servidor Web de pruebas (Plataforma WordPress)[cite: 3] |
| `router.iespoveda.com` | `172.16.0.1` | Interfaz LAN del Firewall OPNsense[cite: 3] |
| `proxmox.iespoveda.com` | `172.16.0.2` | Nodo principal del Hipervisor de Virtualización[cite: 3] |

---

## 4. Auditoría y Análisis de Logs

El *Dashboard* principal de Technitium proporciona una visibilidad completa del estado de la red mediante gráficos estadísticos de tráfico en tiempo real[cite: 3]. Para tareas de análisis forense y auditoría de sistemas, la sección **Query Logs** registra minuciosamente la trazabilidad de cada petición entrante, detallando la fecha, hora, IP de origen, dominio solicitado y la acción aplicada (Permitido / Bloqueado)[cite: 3].

### Caso Práctico de Trazabilidad
Analizando las auditorías del sistema, se puede detectar de forma inmediata el comportamiento anómalo o los intentos de violación de políticas en el aula[cite: 3]. 

* **Ejemplo de registro analizado:** El equipo cliente con la IP `172.16.5.1` (segmento de Alumnos) generó una alerta y su petición fue completamente denegada el día **27/05/2026 a las 10:40:34** al intentar resolver un dominio catalogado como restringido dentro de la lista de bloqueo activa de Steven Black[cite: 3]. El sistema cortó la conexión antes de que se estableciera tráfico de datos hacia el sitio web prohibido[cite: 3].
