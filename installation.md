# Instalación

- [Instalación](#installation)
- [Configuración](#configuration)
	- [Configuración básica](#basic-configuration)
	- [Configuración de entorno](#environment-configuration)
	- [Configuración de caché](#configuration-caching)
	- [Accesando a variables de configuraciones](#accessing-configuration-values)
	- [Nombrando a su aplicación](#naming-your-application)
- [Modo de mantenimiento](#maintenance-mode)

<a name="installation"></a>
## Instalación

### Requerimientos del servidor

El framework Laravel exige algunos requerimientos. Desde luego, todos esos requerimientos vienen preconfigurados en la máquina virtual [Laravel Homestead](/{{version}}/homestead):

<div class="content-list" markdown="1">
- PHP >= 5.5.9
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

<a name="install-laravel"></a>
### Instalar Laravel

Laravel utiliza [Composer](http://getcomposer.org) para administrar sus dependencias. Dicho eso, antes de poder usar Laravel, asegúrese que su máquina tenga Composer instalado.

#### Vía el Instalador Laravel

Primero, descargue el Instalador de Laravel utilizando Composer:

	composer global require "laravel/installer=~1.1"

Asegúrese de agregar el directorio `~/.composer/vendor/bin` a tu variable PATH así, el ejecutable `laravel` puede ser localizado por su sistema.

Una vez instalado, el simple comando `laravel new` creará una nueva instalación de Laravel en el directorio que usted eligió. Por ejemplo, `laravel new blog` crea un directorio llamado `blog` con los archivos para una nueva instalación de Laravel con todas las dependencias ya instaladas. Ese método es mas eficiente que la instalación usando a Composer:

	laravel new blog

#### Vía Composer Create-Project

También puede instalar Laravel con el comando de Composer `create-project` desde su terminal:

	composer create-project laravel/laravel --prefer-dist

<a name="configuration"></a>
## Configuración

<a name="basic-configuration"></a>
### Configuración Básica

Todos los archivos de configuración de Laravel se encuentan en la carpeta `config`. Cada posible opción cuenta con documentación. Asegúrese de eharle un vistazo a esos archivos y familiarizarse con las opciones disponibles.

#### Permisos en los directorio

Después de instalar a Laravel, es posible que necesitará configurar algunos permisos. Los directorios dentro `storage` y los de `bootstrap/cache` deben modificables por el servidor. Si usted está usando la máquina virtual [Homestead](/{{version}}/homestead), esos permisos ya están configurados.

#### Código de la Aplicación

Lo siguiente que tiene que hacer después de la instalación es configurar el código aleatorio de su aplicación. Si instaló Laravel vía Composer o el Instalador de Laravel, el código viene configirado por el comando `key:generate`. Generalmente, el código es una cadena de 32 carácteres. El código puede ser seteado en el archivo de configuración de entorno `.env`. Si todavía usted no ha renombrado el archivo `.env.example` a `.env`, debe hacerlo ahora. **Si la aplicación no cuenta con un código, las sessiones y otras informaciones encriptadas no serán seguras!**

#### Configuraciones adicionales

Generalmente, Laravel no requiere otras configuraciones. A ese punto usted puede empezar a progamar! Sin embargo, quizá querrá revisar  el archivo `config/app.php` y su documentación. Contiene varias opciones como por ejemplo `timezone` y `locale` que usted deberá configurar dependiendo de su aplicación.

Quiźa querrá configurar algunos componente de Laravel, como:

- [Cache](/{{version}}/cache#configuration)
- [Database](/{{version}}/database#configuration)
- [Session](/{{version}}/session#configuration)

Una vez instalado, debe tambíen [configurar su variables de entorno](/{{version}}/installation#environment-configuration).

<a name="pretty-urls"></a>
#### URL amigables

**Apache**

El framework con el archivo `public/.htaccess` que permite ver los URL sin el `index.php`. Si está usando Apache como de Laravel, asegúrese de activar el módulo `mod_rewrite`.

En caso de que el archivo `.htaccess` que viene con Laravel no funciona para usted, puede probar con el siguiente:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

**Nginx**

En Nginx, usted puede utilizar la siguiente directiva para tener URL "amigables":

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

Por supuesto, usando [Homestead](/{{version}}/homestead), las URL amiables serán configuradas automaticamente.

<a name="environment-configuration"></a>
### Configuración de entorno

Generalmente, es útil tener diferentes configuraciones basado en el entorno que su aplicación está corriendo. Por ejemplo, puede tener localmente un controlador de caché diferente al que se está usando en su entorno de produción. Es bastante facil utilizando la configuración basada en el entorno.

Para hacer eso facil como un juego de niño, Laravel utiliza la librería PHP [DotEnv](https://github.com/vlucas/phpdotenv) mantenida por Vance Lucas. En una instalación nueva de Laravel, hay un arcivo en la raíz de su aplicación llamado `.env.example`. Si usted usó Composer para instalar Laravel, ese archivo se renombró a `.env`. Otra manera, tendrá que hacer eso manualmente.

Todas la variables en el archivo `.env` serán cargadas a la variable super-global de PHP `$_ENV` cada vez que su aplicación recibe una consulta http. Puede usar el método corto `env` para recuperar valores de esas variables. De hecho, si usted abre algunos de los archivos de configuración de Larevel, puede ver muchas opciones utilizando ese método corto!

Siéntese seguro de modificar esas variables según las necesidaddes de su aplicación en local o producción. Ten en cuenta que, el `.env` no debe ser subido al control de versión de su aplicación, ya que cada desarrollador / servidor tiene necesidades diferentes en cuanto a configuración del entorno.

Si está desarrollando en equipo, usted querrá incluir un archivo `.env.example`. Al agregar variables libres (place-holder) en el archivo de ejemplo, los otros desarrolladores de su equipo pueden claramente ver cúales son las variables que la aplicación requiera.

#### Accesando al entorno actual de la aplicación

El entorno actual de la aplicacion esta determinado por la variable `APP_ENV` de tu archivo `.env`. Puedes acceder a su valor utilizando el método `environment` que se encuenta en el [facade](/{{version}}/facades)) `App`:

	$environment = App::environment();

También puede pasar argumentos al método `environment` para verificar que el entorno iguale a un valor dado. Si es necesario, puede pasar múltiples valores:

	if (App::environment('local')) {
		// el entorno es local
	}

	if (App::environment('local', 'staging')) {
		// El entorno es local o staging...
	}

También puede utilizar el métdo rápido `app` en lugar del facade:

	$environment = app()->environment();

<a name="configuration-caching"></a>
### Configuración de caché

Para dar a la aplicación un empujoncito en cuanto a la velocidad, debe meter en caché todos sus archivos de configuración usando el comando Artisan `config:cache`. Eso resultará en la combinación de todas las opciones en un sólo archivo que Laravel pueda carga más rápido.

Usted debe adquirir el costumbre de ejecutar el comando `config:cache` como parte de su rutina de despliegue de su aplicación.

<a name="accessing-configuration-values"></a>
### Accesando a variables de configuraciones

Puede facilmente accesar a las variables de configuración de su aplicación usando el método rápido `config`. El valor puede ser accesido con el sintaxis de "punto", que incluye el nombre del archivo y la opción que se desea accesar. Si la opción no existe, se puede setear un valor por defector para ser retornado en lugar de null:

	$value = config('app.timezone');

Para setear el valor de una opción de configuración en tiempo de ejecución, pasamos un arreglo al método rápido `config`:

	config(['app.timezone' => 'America/Chicago']);

<a name="naming-your-application"></a>
### Nombrando a su aplicación

Después de instalar Laravel, es posible que querrá "nombrar" a su aplicación. Por defecto, el directorio `app` directory está bajo el espacio de nombre `App`,y es autocargado por Composer usando [El estándar de autocarga PSR-4](http://www.php-fig.org/psr/psr-4/). Pero, si desea que el espacio de nombre sea el nombre de su aplicación por ejemplo, puede facilmente ejecutar el comando Artisan `app:name`.

Por ejemplo, si su aplicación se llama "Horsefly", usted deberá ejecutar el siguiente comando en la raíz de la aplicación:

	php artisan app:name Horsefly

Renombrar a su aplicación es totalment opcional, y puede perfectamente guardar el espacio de nombre por defecto  `App`.

<a name="maintenance-mode"></a>
## Modo de mantenimiento

Cuando la aplicación se encuentra en modo de mantenimiento, una vista por defecto será retornando para todas la consultas http. Es la manera mas facil de "desabilitar" la aplicación mientras está actualizando o dando mantenimiento. El chequeo de modo de mantenimiento viene incluido en capa de middleware de la aplicación. En modo de mantenimiento, la aplicación dispara la excepción `HttpException` con el código de estado 503.

Para pasar a modo de mantenimiento, solamente ejecute el comando Artisan `down`:

	php artisan down

Para salir del modo de mantemiento, ejecute el comando `up`:

	php artisan up

### Plantilla de Respuesta para modo de Mantenimiento

La plantilla por defecto en modo de manteniento, se encuentra en `resources/views/errors/503.blade.php`.

### Modo de mantenimiento & Colas

En modo de Mantenimiento, [los ejecutadores de colas](/{{version}}/queues) serán manejado. Lo jobs se reanudarán una vez que la aplicación está fuera de mantenimiento.
