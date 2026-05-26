Paso 1: Instalación de Dependencias del Sistema

Antes de descargar WordPress, es necesario preparar el entorno LAMP instalando el servidor web Apache junto con todos los módulos de PHP requeridos para el correcto funcionamiento del CMS:
Bash

sudo apt update
sudo apt install apache2 php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y

Paso 2: Descarga y Preparación de WordPress

Descargamos la última versión oficial de WordPress directamente desde sus servidores oficiales, la descomprimimos en el directorio temporal y la movemos a la ruta pública del servidor:
Bash

cd /tmp
wget [https://wordpress.org/latest.tar.gz](https://wordpress.org/latest.tar.gz)
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/html/theproyect

Paso 3: Configuración de Permisos de Seguridad

Asignamos al usuario de Apache (www-data) como dueño del directorio para permitir que WordPress pueda gestionar la subida de archivos multimedia, instalación de temas y actualizaciones de plugins sin conflictos de permisos:
Bash

sudo chown -R www-data:www-data /var/www/html/theproyect
sudo chmod -R 755 /var/www/html/theproyect

Paso 4: Configuración del Servidor Web (Apache VirtualHost)

Creamos y editamos un archivo de configuración personalizado para el sitio web dentro de los sitios disponibles de Apache:
Bash

sudo nano /etc/apache2/sites-available/theproyect.conf

Añadimos la siguiente estructura de directivas para el hosting virtual:
Apache

<VirtualHost *:80>
    ServerName theproyect
    DocumentRoot /var/www/html/theproyect

    <Directory /var/www/html/theproyect>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/theproyect_error.log
    CustomLog ${APACHE_LOG_DIR}/theproyect_access.log combined
</VirtualHost>

Paso 5: Activación del Sitio y Enrutamiento

Desactivamos el sitio por defecto de Apache para evitar solapamientos y habilitamos la configuración del nuevo proyecto:
Bash

# Comprobar el estado actual del sitio
sudo a2query -s theproyect

# Deshabilitar sitio por defecto y habilitar el nuevo
sudo a2dissite 000-default.conf
sudo a2ensite theproyect.conf

# Recargar configuración del servicio
sudo systemctl reload apache2

Verificamos que el contenido del directorio público sea correcto y mantenga la estructura de archivos nativa de WordPress:
Bash

ls -la /var/www/html/theproyect

Paso 6: Configuración del Archivo de Conexión a la Base de Datos

Accedemos a la carpeta pública del proyecto y generamos el archivo de configuración definitivo (wp-config.php) a partir de la plantilla de ejemplo provista:
Bash

cd /var/www/html/theproyect
sudo cp wp-config-sample.php wp-config.php

Editamos el archivo con los parámetros de conexión correspondientes al servidor de base de datos de nuestra infraestructura:
Bash

sudo nano wp-config.php

Modificamos las siguientes líneas con las credenciales del entorno:
PHP

// ** Ajustes de la base de datos ** //
define( 'DB_NAME', 'aplicacion_web' );
define( 'DB_USER', 'usuario_web' );
define( 'DB_PASSWORD', 'jesus12345' );
define( 'DB_HOST', '10.0.0.13' );
define( 'DB_CHARSET', 'utf8mb4' );

Paso 7: Reinicio y Verificación del Sistema

Reiniciamos por completo el servicio de Apache para asegurar que todos los módulos y directivas se carguen limpiamente, y comprobamos que el estado del servicio sea activo (running):
Bash

sudo systemctl restart apache2
sudo systemctl status apache2

🌐 Asistente de Instalación Web

Una vez que el backend está listo, abrimos el navegador e ingresamos a la dirección IP o dominio configurado (ej: http://172.16.5.100:8080/wp-admin/install.php).

    Selección de Idioma: Elegimos el idioma deseado para el panel de administración y hacemos clic en continuar.

    Información del Sitio: Completamos los datos de administración de la plataforma (Título del sitio, nombre del usuario administrador, contraseña y correo electrónico de contacto) y ejecutamos la instalación final.
