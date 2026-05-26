# Guía de Instalación de Servidor WordPress Paso a Paso

## Paso 1: Instalación de Dependencias del Sistema

Antes de descargar WordPress, es necesario preparar el entorno **LAMP** instalando el servidor web **Apache** junto con todos los módulos de **PHP** requeridos para el correcto funcionamiento del CMS.

### Instalación de Apache y PHP

```bash
sudo apt update
sudo apt install apache2 php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
```

---

## Paso 2: Descarga y Preparación de WordPress

Descargamos la última versión oficial de **WordPress** directamente desde sus servidores oficiales, la descomprimimos en el directorio temporal y la movemos a la ruta pública del servidor.

### Descargar WordPress

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz
sudo mv wordpress /var/www/html/theproyect
```

---

## Paso 3: Configuración de Permisos de Seguridad

Asignamos al usuario de Apache (`www-data`) como propietario del directorio para permitir que WordPress pueda gestionar:

- Subida de archivos multimedia
- Instalación de temas
- Actualizaciones de plugins

Sin conflictos de permisos.

### Configurar permisos

```bash
sudo chown -R www-data:www-data /var/www/html/theproyect
sudo chmod -R 755 /var/www/html/theproyect
```

---

## Paso 4: Configuración del Servidor Web (Apache VirtualHost)

Creamos y editamos un archivo de configuración personalizado para el sitio web dentro de los **sitios disponibles de Apache**.

### Crear el archivo de configuración

```bash
sudo nano /etc/apache2/sites-available/theproyect.conf
```

### Configuración del VirtualHost

Añadimos la siguiente estructura de directivas para el hosting virtual:

```apache
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
```

---

## Paso 5: Activación del Sitio y Enrutamiento

Desactivamos el sitio por defecto de Apache para evitar conflictos y habilitamos la configuración del nuevo proyecto.

### Comprobar el estado actual del sitio

```bash
sudo a2query -s theproyect
```

### Deshabilitar el sitio por defecto y habilitar el nuevo

```bash
sudo a2dissite 000-default.conf
sudo a2ensite theproyect.conf
```

### Recargar la configuración de Apache

```bash
sudo systemctl reload apache2
```

### Verificar la estructura de archivos de WordPress

Comprobamos que el contenido del directorio público sea correcto y mantenga la estructura nativa de WordPress.

```bash
ls -la /var/www/html/theproyect
```

---
antes de continuar, tendremos que haber configurado el servidor de base de datos
# PARTE 2: Configuración del Servidor de Base de Datos (MySQL)

Una vez preparado el servidor web, configuraremos el **servidor de base de datos MySQL**, el cual almacenará toda la información de WordPress (usuarios, entradas, configuración, plugins, etc.).

## Paso 1: Instalación de MySQL Server

Primero instalamos el servicio de **MySQL Server** en el servidor destinado a la base de datos.

### Instalar MySQL

```bash
sudo apt update
sudo apt install mysql-server -y
```

### Verificar el estado del servicio

Comprobamos que el servicio se encuentre activo y funcionando correctamente:

```bash
sudo systemctl status mysql
```

El resultado esperado debe indicar:

```text
Active: active (running)
```

---

## Paso 2: Configuración de Acceso Remoto de MySQL

Por defecto, MySQL solo escucha conexiones locales (`127.0.0.1`). Como WordPress estará en otro servidor, debemos modificar la configuración para permitir conexiones desde la red.

### Editar el archivo de configuración

Abrimos el archivo principal de configuración de MySQL:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

### Modificar la IP de escucha (`bind-address`)

Buscamos la siguiente línea:

```ini
bind-address = 127.0.0.1
```

Y la modificamos por la IP del servidor de base de datos:

```ini
bind-address = 10.0.0.13
```

> **Nota:** Sustituye `10.0.0.13` por la dirección IP real de tu servidor MySQL.

Esto permitirá que el servidor WordPress pueda localizar y conectarse correctamente a la base de datos.

### Reiniciar MySQL

Después de aplicar cambios, reiniciamos el servicio:

```bash
sudo systemctl restart mysql
```

---

## Paso 3: Creación de la Base de Datos y Usuario

Accedemos al gestor de MySQL como administrador (`root`):

```bash
sudo mysql
```

### Crear la base de datos

Creamos una base de datos para WordPress:

```sql
CREATE DATABASE aplicacion_web;
```

### Crear un usuario para WordPress

Creamos un usuario específico para la aplicación web:

```sql
CREATE USER 'usuario_web'@'%' IDENTIFIED BY 'jesus12345';
```

> El símbolo `%` permite conexiones desde cualquier host. En entornos reales es recomendable limitarlo a la IP del servidor WordPress.

### Asignar permisos sobre la base de datos

Concedemos permisos completos sobre la base de datos creada:

```sql
GRANT ALL PRIVILEGES ON aplicacion_web.* TO 'usuario_web'@'%';
```

### Aplicar cambios de privilegios

```sql
FLUSH PRIVILEGES;
```

### Salir de MySQL

```sql
EXIT;
```

---

## Paso 4: Verificación de la Base de Datos

Volvemos a entrar en MySQL para comprobar que la base de datos se ha creado correctamente.

### Acceder a MySQL

```bash
sudo mysql
```

### Mostrar bases de datos existentes

```sql
SHOW DATABASES;
```

Si todo ha funcionado correctamente, debería aparecer una salida similar a esta:

```text
+--------------------+
| Database           |
+--------------------+
| aplicacion_web     |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

Con esto verificamos que la base de datos de WordPress ya está creada y disponible para ser utilizada por el servidor web.

---

## Relación con WordPress

Estos datos deben coincidir exactamente con la configuración del archivo `wp-config.php` del servidor web:

```php
define( 'DB_NAME', 'aplicacion_web' );
define( 'DB_USER', 'usuario_web' );
define( 'DB_PASSWORD', 'jesus12345' );
define( 'DB_HOST', '10.0.0.13' );
```

Si alguno de estos parámetros no coincide, WordPress mostrará un error de conexión a la base de datos.




## Paso 6: Configuración del Archivo de Conexión a la Base de Datos

Accedemos a la carpeta pública del proyecto y generamos el archivo de configuración definitivo (`wp-config.php`) a partir de la plantilla de ejemplo proporcionada por WordPress.

### Crear el archivo de configuración

```bash
cd /var/www/html/theproyect
sudo cp wp-config-sample.php wp-config.php
```

### Editar el archivo de configuración

```bash
sudo nano wp-config.php
```

### Configurar credenciales de la base de datos

Modificamos las siguientes líneas con los parámetros de conexión correspondientes al entorno:

```php
// ** Ajustes de la base de datos ** //
define( 'DB_NAME', 'aplicacion_web' );
define( 'DB_USER', 'usuario_web' );
define( 'DB_PASSWORD', 'jesus12345' );
define( 'DB_HOST', '10.0.0.13' );
define( 'DB_CHARSET', 'utf8mb4' );
```

> **Importante:** Sustituye estos valores por los correspondientes a tu infraestructura si son diferentes.

---

## Paso 7: Reinicio y Verificación del Sistema

Reiniciamos completamente el servicio de Apache para asegurar que todos los módulos y directivas se carguen correctamente.

### Reiniciar Apache

```bash
sudo systemctl restart apache2
```

### Verificar el estado del servicio

```bash
sudo systemctl status apache2
```

Si todo ha funcionado correctamente, el estado debe aparecer como:

```text
Active: active (running)
```

---

# 🌐 Asistente de Instalación Web de WordPress

Una vez que el backend está completamente configurado, abrimos el navegador y accedemos a la dirección IP o dominio configurado.

### Ejemplo de acceso

```text
http://172.16.5.100:8080/wp-admin/install.php
```

## 1. Selección de Idioma

Seleccionamos el idioma deseado para el panel de administración de WordPress y pulsamos en **Continuar**.

## 2. Configuración del Sitio

Completamos los datos de administración de la plataforma:

- **Título del sitio**
- **Usuario administrador**
- **Contraseña**
- **Correo electrónico de contacto**

Una vez completados los datos, hacemos clic en **Instalar WordPress** para finalizar el proceso.

---

## Verificación Final

Si la instalación se ha realizado correctamente, podremos acceder al panel de administración desde:

```text
http://TU_IP/wp-admin
```

O bien usando el dominio configurado en Apache.

```text
http://theproyect/wp-admin
```
