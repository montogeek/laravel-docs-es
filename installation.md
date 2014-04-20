# Instalación

- [Instalar Composer](#install-composer)
- [Instalar Laravel](#install-laravel)
- [Requerimientos del servidor](#server-requirements)
- [Configuración](#configuration)
- [URLs amigables](#pretty-urls)

<a name="install-composer"></a>
## Instalar Composer

Laravel utiliza [Composer](http://getcomposer.org) para administrar sus dependencias. Primero, descarga una copia de `composer.phar`, una vez lo tengas puedes dejarlo en la carpeta de tu proyecto o moverlo a `usr/local/bin` para ejecutarlo de forma global. En Microsoft Windows, puedes usar Composer a través de su [instalador](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Instalar Laravel

### A través del Instalador de Laravel

Primero, descarga el [instalador PHAR de Laravel](http://laravel.com/laravel.phar). Por cuestiones de facilidad, cambie el nombre del archivo a `laravel` y muevelo a `/usr/local/bin`. Una vez instalado, simplemente ejecuta `laravel new` y tendrás una nueva y fresca instalación de Laravel. Si deseas especificar un directorio para instalar Laravel agrega el nombre al final del comando, por ejemplo `laravel new blog` instalará Laravel en el directorio `blog`. Este método de instalación es mucho más rápido que hacerlo a través de Composer.

### A través del Instalador de Composer

Tal vez te gustaría instalar Laravel usando el comando de Composer `create-project` en tu terminal:

    composer create-project laravel/laravel --prefer-dist

### A través de descarga directa

Una vez tengas instalado Composer, descarga la [última versión](https://github.com/laravel/laravel/archive/master.zip) de Laravel y extrae su contenido en algún directorio en tu servidor. A continuación, en la raíz del proyecto, ejecuta el comando `php composer.phar install` (o `composer install`) para instalar todas las dependencias del Framework. Este proceso requiere tener Git instalado en el servidor para hacerlo correctamente.

Si deseas actualizar Laravel, ejecuta el comando `php composer.phar update`.

<a name="server-requirements"></a>
## Requerimientos del servidor

Laravel tiene pocos requerimientos para funcionar correctamente:
- PHP >= 5.3.7
- Extensión MCrypt PHP

Desde la versión 5.5 de PHP, algunas distribuciones de sistemas operativos podrián requerir la instalación manual de la extensión PHP JSON. Si usas Ubuntu, esta extensión se puede instalar con el comando `apt-get install php5-json`.

<a name="configuration"></a>
## Configuración

Laravel practicamente no necesita configuración para funcionar de inmediato. ¡Puedes empezar a programar ahora mismo! Aunque, podrías echarle un vistazo al archivo `app/config/app.php` y leer su documentación, contiene varias opciones como `timezone` y `locale`, las cuales seguramente desearías cambiar para tu aplicación.

<a name="permissions"></a>
### Permisos

Laravel podría requerir la configuración de algunos permisos: las carpetas dentro de `app/storage` requieren permisos de escritura por el servidor Web.

<a name="paths"></a>
### Direcciones

Varias direcciones de los directorios del framework son configurables. Para cambiar la ubicación de dichos directorios revisa el archivo `bootstrap/paths.php`.

<a name="pretty-urls"></a>
## URLs amigables

### Servidor Apache

Laravel posee de forma predeterminada el archivo `public/.htaccess` que es usado para permitir URLs sin `index.php`. Si usas Apache como servidor de tu aplicación de Laravel, asegúrate de habilitar el modulo `mod_rewrite`.

Si el archivo `.htaccess` que trae Laravel no funciona con tu configuración de Apache, prueba este:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Servidor Nginx

Con Nginx, la siguiente directiva en la configuración de tu sitio permitirán URLS amigables.

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }