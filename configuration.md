# Configuración

- [Introducción](#introduction)
- [Después de instalar](#after-installation)
- [Acceder a las opciones de configuración](#accessing-configuration-values)
- [Configuración de entorno](#environment-configuration)
- [Proteger configuración sensible](#protecting-sensitive-configuration)
- [Modo mantenimineto](#maintenance-mode)
- [URLs amigables](#pretty-urls)

<a name="introduction"></a>
## Introducción

Todos los archivos de configuracion de Laravel estan guardados en en el directorio `config`. Cada opcion esta documentada, sientete a gusto de explorar los archivos y familiarizarte con las opciones disponibles.

<a name="configuration"></a>
## Después de instalar

### Nombrando tu aplicación

Despues de instalar Laravel, podras querer "nombrar" tu aplicacion. Por defecto, la carpeta `app` esta nombrada como `App` y autocargada por Composer usando el [estandar de autocargar PSR-4](http://www.php-fig.org/psr/psr-4/). Sin embargo, puedes cambiar el namespace para que concuerde con el nombre de tu aplicación, lo puedes hacer facilmente con el comando de Artisan `app:name`.

Por ejemplo, si tu aplicacion se llamada "Libelula", puedes ejecutar el siguiente comando desde la razi de tu instalacion:

	php artisan app:name Libelula

Renombrar tu aplicacion es completamente opcional, eres libre de mantener el nombre `App` si asi lo deseas.

### Configuracion adicional

Laravel necesita muy poca configuracion para funcionar. Puedes empezar a programar! Sin embargo, deseas revisar el archivo `config/app.php` y su documentacion. Este contiene varias opciones como `timezone` y `locale` que desearas cambiar segun tu ubicacion.

Una vez Laravel este instalado, tambien puedes [configurar tu entorno local](/5.0/configuration#environment-configuration).

> **Nota:** Jamas deberias tener la opcion de configuracion `app.debug` en `true` para la aplicacion en el entorno de produccion.

<a name="permissions"></a>
### Permisos

Laravel puede requerir varios permisos para funcionar: Los directorios dentro de `storage` requieren permisos de acceso por el servidor web.

<a name="accessing-configuration-values"></a>
## Acceder a las opciones de configuración

Puedes acceder facilemente a tu configuracion usando el facade `Config`:

	$value = Config::get('app.timezone');

	Config::set('app.timezone', 'America/Chicago');

Tambien puedes usar la funcion de ayuda `config`:

	$value = config('app.timezone');

<a name="environment-configuration"></a>
## Configuración de entorno

Es muy util tener diferente configuraciones segun el entorno en el que este corriendo nuestra aplicacion. Por ejemplo, puedes necesitar usar un controlador de cache diferente para tu entorno local y otro para el entorno de produccion. Es muy facil usar configuracion basada en entornos.

Para hacer esto mas facil, Laravel utiliza la libreria PHP [DotEnv](https://github.com/vlucas/phpdotenv) de Vance Lucas. En una nueva instalacion de Laravel, la carpeta raiz del proyecto tendra un archivo `.env.example`. Si instalaste Laravel a traves de Composer, este archivo automaticamente se renombrara a `.env`. De otro modo, deberias renombrar el archivo manualmente.

Todas las variables listadas en este archivo seran cargadas en la variable super global de php `$_ENV` cuando tu aplicacion reciba una peticion. Puedes usar el metodo de ayuda `env` para obtener los valores de esas variables. De hecho, si revisaste los archivos de configuracion de Laravel, te diste cuenta que algunas opciones usan este metodo de ayuda!.

Se libre de modificar tus variables de entorno como lo necesites para tu servidor local, asi como para tu servidor de produccion. Sin embargo, tu archivo `.env` no deberia ser incluido en tu sistema de control de versiones, dado que cada desarrollador/servidor usando tu aplicacion podria necesitar una configuracion diferente.

Si estas desarrollando con un equipo de trabajo, podrias necesitar incluir un archivo `.env.example` en tu aplicacion. Estableciendo valores de ejemplo en los archivos de configuracion le ayudara a otros desarrolladores de tu equipo que variables on necesarias para ejecutar tu aplicacion.

#### Acceder al entorno actual de la aplicacion

Puedes acceder al entorno actual de tu aplicacion a traves del metodo `environment` de la instancia `Application`:

	$environment = $app->environment();

Tambien puedes pasar argumentos al metodo `environment` para verificar si el entorno concuerca con el valor dado:

	if ($app->environment('local'))
	{
		// The environment is local
	}

	if ($app->environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

Para obtener una instancia de la aplicacion, resuelve el contrato `Illuminate\Contracts\Foundation\Applicacion` a traves del [servicio de contenedores](/5.0/container). Por supuesto, si estas dentro de un [proveedor de servicios](/5.0/providres), la instancia de la aplicacion esta disponible a traves de la variable de la instancia `$this->app`.

Una instancia de la aplicación tambien puede ser accedida a traves del metodo `app` or el facade `App`:

	$environment = app()->environment();

	$environment = App::environment();

<a name="maintenance-mode"></a>
## Modo mantenimineto

Cuando tu aplicacion esta en modo mantenimiento, una vista personalizada sera mostrada para todas tus peticiones. Esto hace muy facil "deshabilitar" tu aplicacion mientras se esta actualizando o cuando estas haciendo un mantenimiento. Un verificador de modo mantenimiento esta incluido de forma predeterminada en el middleware(nota) de tu aplicacion. Si la aplicacion esta en modo mantenimiento una excepcion `HtppException` sera enviada con un codigo de estado 503.

Para habilitar el modo mantenimiento, simplemente ejecuta el comando de Artisan `down`:

	php artisan down

Para deshabilitar el modo mantenimiento, usa el comando `up`:

	php artisan up

### Plantilla del modo en mantenimiento

La plantilla o vista predeterminada para las respuesta en modo mantenimiento esta guardada en `resources/templates/errors/503.blade.php`.

### Modo mantenimiento y Colas

Mientras tu aplicacion esta en modo mantenimiento, ningun [trabajo encolado](/5.0/queues) sera ejecutado. Los trabajos continuaran ejecutandose normalmente una vez la aplicacion no este en modo mantenimiento.

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
