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

### Equipo en la red de alumnado
Un servidor dedicado que proporciona:

- **FOG**: despliegue y restauración de equipos sin necesidad de pendrive.
- **RADIUS**: autenticación centralizada.

### Patch panel
Para el aula de 1º de Asir.

### Conectores rj45 machos y hembras
Para las aulas que lo requieran.
