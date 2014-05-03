# Configuration

- [Introducción](#introduction)
- [Configuración de entorno](#environment-configuration)
- [Configuración de proveedores](#provider-configuration)
- [Protegiendo configuración sensible](#protecting-sensitive-configuration)
- [Modo mantenimiento](#maintenance-mode)

<a name="introduction"></a>
## Introducción

Todos lo archivos de configuración de Laravel están guardados en el directorio `app/config`. En todos los archivos cada opción de configuración está documentada, así que siéntete a gusto conociendo todas las opciones que tienes disponible.

Algunas veces necesitar acceder a las opciones de configuración. Puedes realizarlo haciendo uso de la clase `Config`:

#### Acceder a una opción de configuración

	Config::get('app.timezone');

Puedes también espeficar un valor predeterminado para retornar y la opción de configuración no existe:

	$timezone = Config::get('app.timezone', 'UTC');

Observa que la síntaxis de "puntos" puede ser usada para acceder a las opciones de configuración en múltiples archivos. También puedes establecer opciones de configuración durante la ejecución:

#### Establecer una opción de configuración

	Config::set('database.default', 'sqlite');

Las opciones de configuración que son establecidas durante la ejecución solo son establecidas durante la petición actual, si no se tendrán en cuenta en las siguiente peticiones.

<a name="environment-configuration"></a>
## Configuración de entorno

A menudo es muy útil tener valores diferentes de configuración según el entorno en el cual se esté ejecutanddo la aplicación. Por ejemplo, tal vez desees utilizar un controlador de cache diferente en tu entorno de desarrollo local y otro en el servidor de producción. Esto es muy fácil de hacer usando configuraciones basadas en entornos.

Simplemente crea una carpeta con un nombre que sea igual a tu entorno de desarrollo dentro en el directorio `config`, por ejemplo `local`. Después, crea los archivos de configuración que desees sobreescribir y especifica las opciones para tu entorno. Por ejemplo, para sobreescribir la configuración de cache para el entorno local, creas el archivo `cache.php` en `app/config/local` con el siguiente contenido:

	<?php

	return array(

		'driver' => 'file',

	);

> **Nota:** No uses 'testing' como nombre para tu entorno. Está reservado para las pruebas unitarias.

Observa que no es necesario especificar _cada_ opción disponible en el archivo de configuración predeterminado, solamente las opciones que desees sobreescribir. Los archivos de configuración de tu entorno se aplicarán en "cascada" sobre los archivos predeterminados.

A continuación, necesitamos decirle al framework como determinar en que entorno se está ejecutando. El entorno predeterminado siempre es `production`. sin embargo, tal vez necesites configurar otro entornos en el archivo `boostrap/start.php` en la raíz de tu instalación de Laravel. En este archivo te encontrarás con una llamada a `$app->detectEnvironment`. El arreglo pasado a este método es usado para determinar el entorno de ejecución actual. Puedes agregar otros entornos y nombres de máquinas al arreglo como desees.

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

En este ejemplo, 'local' es el nombre del entorno y 'your-machine-name' es el nombre de la máquina en tu servidor. En Linux, Mac y Windows, puedes saber el nombre de tu máquina ejecutando el comando `hostname` en la terminal .

Si necesitas detectar de manera más flexible el entorno, puedes pasar una Clausura al método `detectEnvironment`, permitiéndote implementar tu propia forma de detectar el entorno:

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

Puedes acceder al entorno actual de la aplicación a través del método `environment`.

#### Accediendo al entorno actual de la aplicación

	$environment = App::environment();

You may also pass arguments to the `environment` method to check if the environment matches a given value:

	if (App::environment('local'))
	{
		// The environment is local
	}

	if (App::environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

<a name="provider-configuration"></a>
### Configuración de proveedores

When using environment configuration, you may want to "append" environment [service providers](/docs/ioc#service-providers) to your primary `app` configuration file. However, if you try this, you will notice the environment `app` providers are overriding the providers in your primary `app` configuration file. To force the providers to be appended, use the `append_config` helper method in your environment `app` configuration file:

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## Protegiendo configuración sensible

For "real" applications, it is advisable to keep all of your sensitive configuration out of your configuration files. Things such as database passwords, Stripe API keys, and encryption keys should be kept out of your configuration files whenever possible. So, where should we place them? Thankfully, Laravel provides a very simple solution to protecting these types of configuration items using "dot" files.

First, [configure your application](/docs/configuration#environment-configuration) to recognize your machine as being in the `local` environment. Next, create a `.env.local.php` file within the root of your project, which is usually the same directory that contains your `composer.json` file. The `.env.local.php` should return an array of key-value pairs, much like a typical Laravel configuration file:

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

All of the key-value pairs returned by this file will automatically be available via the `$_ENV` and `$_SERVER` PHP "superglobals". You may now reference these globals from within your configuration files:

	'key' => $_ENV['TEST_STRIPE_KEY']

Be sure to add the `.env.local.php` file to your `.gitignore` file. This will allow other developers on your team to create their own local environment configuration, as well as hide your sensitive configuration items from source control.

Now, On your production server, create a `.env.php` file in your project root that contains the corresponding values for your production environment. Like the `.env.local.php` file, the production `.env.php` file should never be included in source control.

> **Note:** You may create a file for each environment supported by your application. For example, the `development` environment will load the `.env.development.php` file if it exists.

<a name="maintenance-mode"></a>
## Modo mantenimiento

When your application is in maintenance mode, a custom view will be displayed for all routes into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A call to the `App::down` method is already present in your `app/start/global.php` file. The response from this method will be sent to users when your application is in maintenance mode.

To enable maintenance mode, simply execute the `down` Artisan command:

	php artisan down

To disable maintenance mode, use the `up` command:

	php artisan up

To show a custom view when your application is in maintenance mode, you may add something like the following to your application's `app/start/global.php` file:

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});

If the Closure passed to the `down` method returns `NULL`, maintenace mode will be ignored for that request.

### Maintenance Mode & Queues

While your application is in maintenance mode, no [queue jobs](/docs/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.
