# Instalación

- [Instalar Composer](#install-composer)
- [Instalar Laravel](#install-laravel)
- [Requerimientos del servidor](#server-requirements)
- [Configuración](#configuration)
- [URLs amigables](#pretty-urls)

<a name="install-composer"></a>
## Install Composer

Laravel utiliza [Composer](http://getcomposer.org) para administrar sus dependencias. Primero, descarga una copia de `composer.phar`, una vez lo tengas puedes dejarlo en la carpeta de tu proyecto o moverlo a `usr/local/bin` para ejecutarlo de forma global. En Microsoft Windows, puedes usar Composer a través de su [instalador](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Instalar Laravel

### A través del Instalador de Laravel

Primero, descarga el [instalador PHAR de Laravel](http://laravel.com/laravel.phar). Por cuestiones de facilidad, cambie el nombre del archivo a `laravel` y muevelo a `/usr/local/bin`. Una vez instalado, simplemente ejecuta `laravel new` y tendrás una nueva y fresca instalación de Laravel. Si deseas especificar un directorio para instalar Laravel agrega el nombre al final del comando, por ejemplo `laravel new blog` instalará Laravel en el directorio `blog`. Este método de instalación es mucho más rápido que hacerlo a través de Composer.

### A través del Instalador de Composer

You may also install Laravel by issuing the Composer `create-project` command in your terminal:

	composer create-project laravel/laravel --prefer-dist

### A través de descarga directa

Once Composer is installed, download the [latest version](https://github.com/laravel/laravel/archive/master.zip) of the Laravel framework and extract its contents into a directory on your server. Next, in the root of your Laravel application, run the `php composer.phar install` (or `composer install`) command to install all of the framework's dependencies. This process requires Git to be installed on the server to successfully complete the installation.

If you want to update the Laravel framework, you may issue the `php composer.phar update` command.

<a name="server-requirements"></a>
## Requerimientos del servidor

The Laravel framework has a few system requirements:

- PHP >= 5.3.7
- MCrypt PHP Extension

As of PHP 5.5, some OS distributions may require you to manually install the PHP JSON extension. When using Ubuntu, this can be done via `apt-get install php5-json`.

<a name="configuration"></a>
## Configuración

Laravel needs almost no configuration out of the box. You are free to get started developing! However, you may wish to review the `app/config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

<a name="permissions"></a>
### Permisos
Laravel may require one set of permissions to be configured: folders within app/storage require write access by the web server.

<a name="paths"></a>
### Direcciones

Several of the framework directory paths are configurable. To change the location of these directories, check out the `bootstrap/paths.php` file.

<a name="pretty-urls"></a>
## URLs amigables

### Servidor Apache

The framework ships with a `public/.htaccess` file that is used to allow URLs without `index.php`. If you use Apache to serve your Laravel application, be sure to enable the `mod_rewrite` module.

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this one:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Servidor Nginx

On Nginx, the following directive in your site configuration will allow "pretty" URLs:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }