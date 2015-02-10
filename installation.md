# Instalación

- [Instalar Composer](#install-composer)
- [Instalar Laravel](#install-laravel)
- [Requerimientos del servidor](#server-requirements)

<a name="install-composer"></a>
## Instalar Composer

Laravel utiliza [Composer](http://getcomposer.org) para administrar sus dependencias. Antes de usar Laravel, asegúrate de tener instalado Composer en tu sistema.

<a name="install-laravel"></a>
## Instalar Laravel

### Instalador de Laravel

Primero, descarga el instalador de Laravel utilizando Composer.

	composer global require "laravel/installer=~1.1"

Asegúrate de agregar el directorio `~/.composer/vendor/bin` a tu variable de entorno PATH, así tu sistema podrá localizar el archivo ejecutable `laravel`.

Una vez instalado, el comando `laravel new` creará una nueva instalación de Laravel en el directorio especificado. Por ejemplo, `laravel new blog` creará un directorio llamado `blog`, el cual contiene una nueva instalación de Laravel con todas sus dependencias. Este método de instalación es mucho más rápido que instalar Laravel utilizando Composer:

	laravel new blog

### Composer Create-Project

Puedes instalar Laravel usando el comando `create-project` de Composer en tu terminal:

	composer create-project laravel/laravel --prefer-dist

<a name="server-requirements"></a>
## Requerimientos del servidor

Laravel tiene algunos requerimientos:

- PHP >= 5.4
- Extensión Mcrypt PHP
- Extensión OpenSSL PHP
- Extensión Mbstring PHP

En PHP 5.5, algunos sistemas operativos requieren que instales manualmente la extensión PHP JSON. Si usas Ubuntu, lo puedes instalar con el comando `apt-get install php5-json`.

<a name="configuration"></a>
## Configuración

Lo primero que debes hacer despues de instalar Laravel es establecer la llave de tu aplicacion. Si instalaste Laravel a traves de Composer, esta llave probablemente ya fue establecida por ti por el comando `key:generate`.

Normalmente, esta cadena debe tener una longitud de 32 caracteres. Esta llave puede ser establecida en el archivo de configuracion `config/app.php`. **Si la llave no establecida, las sesiones de usuarios y otra informacion encriptada no sera segura!**

Laravel no necesita mas configuracion para funcionar. Eres libre para empezar a desarrollar! Sin embargo, puedes revisar el archivo `config/app.php` y su documentacion, contiene multiples opciones como `timezone` y `locale` que seguro necesitaras cambiar de acuerdo a tu aplicacion.

Una vez Laravel este instalado, puedes [configurar tu entorno local](/5.0/configuration#environment-configuration).

> **Nota:** Jamas deberias establecer la opcion `app.debug` en `true` para tu aplicacion en produccion.

<a name="permissions"></a>
### Permisos

Laravel podria requerir permisos para ser configurado: las carpetas de `storage` requieren permisos de escritura por el servidor web.

<a name="pretty-urls"></a>
## URLs amigables

### Apache

El framework viene con un archivo `public/.htaccess` que es usado para permitir URLs sin el prefijo `index.php`. Si usas Apache para servir tu aplicacion, asegurate de tener habilitado el modulo `mod_rewrite`.

Si el archivo `.htaccess` que viene con Laravel no funciona con tu instalacion de Apache, prueba con este:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

Con Nginx, la siguiente directiva en la configuracion de tu sitio permitira URLS "bonitas":

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Por supuesto, usando [Homestead](/5.0/homestead) obtendras URLs bonitas automaticamente.