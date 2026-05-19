# **Trabajo realizado**

En este apartado se resumen las actividades principales: rehabilitación del cableado de red del 2.º curso de ASIR e implementación del servidor FOG.

## Rehabilitación de la red

- Se inspeccionaron todos los conectores y latiguillos del switch, comprobando continuidad, polaridad y calidad de las terminaciones.
- Se reterminaron y repararon 10 conectores en los puertos 1, 5, 6, 7, 13, 17, 19, 20, 21 y 23. Se reorganizó el tendido, se etiquetaron las canaletas y se documentó la intervención para facilitar el mantenimiento.

Figuras representativas del trabajo de rehabilitación:

### Comprobación en canaletas (vistas)

- Uso de tester para comprobación de continuidad
![Uso de tester para comprobación de continuidad](imagenes/IMG_6631.jpeg)

- Puertos 1 y 2 en canaleta
![Puertos 1 y 2 en canaleta](imagenes/IMG_6633.jpeg)

- Puertos 5 y 6 en canaleta
![Puertos 5 y 6 en canaleta](imagenes/IMG_6632.jpeg)


### Intervención en el backpanel (vistas relacionadas)

- Reparación y reterminado en puertos defectuosos
![Reparación y reterminado en puertos defectuosos](imagenes/IMG_6619.jpeg)

- Cableado del backpanel tras la intervención
![Cableado del backpanel tras la intervención](imagenes/IMG_6620.jpeg)

Etiquetado de puertos en canaletas
![Etiquetado de puertos en canaletas](imagenes/IMG_6621.jpeg)

Reparación de conectores en los extremos
![Reparación de conectores en los extremos](imagenes/IMG_6622.jpeg)

Fabricación de latiguillos a medida para mejorar la presentación
![Fabricación de latiguillos a medida para mejorar la presentación](imagenes/IMG_6629.jpeg)

## 2. Implementación del servidor FOG

- Se instaló Ubuntu 26.04 LTS en la máquina destinada a servidor; se aplicaron configuraciones básicas de seguridad (actualizaciones, firewall y cuentas de servicio) y ajustes de red (IP estática, DNS).
- Se instaló y configuró FOG, verificando la comunicación con máquinas cliente y realizando pruebas iniciales de creación y despliegue de imágenes.

Investigación y configuración inicial del servidor FOG
![Investigación y configuración inicial del servidor FOG](imagenes/IMG_6628.jpeg)

En la puesta en marcha nos hemos encontrado con varios problemas:
![Reading partition Tables..... Failed](imagenes/IMG_6634.jpeg)

Este problema ha ocurrido ya que FOG por defecto intenta hacer la copia del primer disco que encuentra, al activar el modo debug en la tarea:
![Activación del modo debug](imagenes/IMG_6636.jpg)

 y poner el comando lsblk nos sale lo siguiente:

 ![Listado de discos](/imagenes/IMG_6637.jpg)

 Como vemos el primer disco es donde se guardan los datos y el segundo es donde se guarda el sistema operativo. Necesitamos indicar esto en el servidor FOG:

 ![Host Primary Disk](/imagenes/IMG_6638.jpg)

 Indicamos en el campo Host Primary Disk el disco del que queremos hacer la imagen.

## 3. Instalación de Anydesk en servidor FOG
Realizamos la instalación:

`wget --max-redirect 1 --trust-server-names 'https://anydesk.com/en/downloads/thank-you?dv=deb_64' -O anydesk.deb &&
sudo apt install ./anydesk.deb
`

Una vez entramos y intentamos conectarnos desde otro equipo saldra este error, ya que Anydesk no es compatible con el sistema Wayland:

![Error por Wayland](/imagenes/IMG_6640.jpg)


Ejecutamos 

`
sudo apt update &&
sudo apt install xfce4 xfce4-goodies lightdm -y &&  sudo dpkg-reconfigure gdm3
`

En el desplegable seleccionaremos lightdm.

Al iniciar sesión deberemos seleccionar Sesión de xfce:

![Sesión de Xfce](/imagenes/IMG_6641.jpg)

### Otras configuraciónes de Anydesk

#### Acceso por contraseña

Accedemos a Anydesk -> Configuración -> acceso -> Desbloquear el control de seguridad -> Cambiar la contraseña de este puesto de trabajo (Le asigne la contraseña "**2Asir**"), además le configuramos una restricción de acceso para evitar accesos indevidos y excluimos este equipo de descubrimientos.

![Asignar contraseña para acceso no presencial y restricción de acceso](/imagenes/IMG_6643.jpg)

![Excluir el dispositivo de descubrimiento](/imagenes/IMG_6644.jpg)

Conclusión

Las actuaciones ejecutadas garantizan una infraestructura de red operativa y un servidor de despliegue preparado para la gestión de imágenes y la automatización de instalaciones, facilitando futuras ampliaciones y el mantenimiento del laboratorio.