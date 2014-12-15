# Configuration

- [Introducción](#introduction)
- [Configuración del Entorno](#environment-configuration)
- [Configuración de Proveedores](#provider-configuration)
- [Protegiendo Configuración Sensible](#protecting-sensitive-configuration)
- [Modo Mantenimiento](#maintenance-mode)

<a name="introduction"></a>
## Introducción

Todos los archivos de configuración de Laravel están guardados en el directorio `app/config`. En todos los archivos cada opción de configuración está documentada, así que siéntete a gusto de ver los archivos y familiarizarte con las opciones que tienes disponible.

Algunas veces puedes necesitar acceder a las opciones de configuración durante la ejecución. Puedes realizarlo usando la clase `Config`:

#### Acceder a una Opción de Configuración

	Config::get('app.timezone');

También puedes espeficar un valor predeterminado para retornar si la opción de configuración no existe:

	$timezone = Config::get('app.timezone', 'UTC');

Nótese que el estilo de sintáxis con "puntos" puede ser usada para acceder a las opciones de configuración en múltiples archivos. También puedes establecer opciones de configuración durante la ejecución:

#### Establecer una Opción de Configuración

	Config::set('database.default', 'sqlite');

Las opciones de configuración que son establecidas en tiempo de ejecución solo son establecidas para la petición actual, y no se tendrán en cuenta en las siguiente peticiones.

<a name="environment-configuration"></a>
## Configuración del Entorno

A menudo es muy útil tener diferentes opciones de configuración basado en el entorno en el cual la aplicación se está ejecutanddo. Por ejemplo, tal vez desees utilizar un controlador de cache diferente en tu entorno de desarrollo local y otro en el servidor de producción. Es fácil de hacerlo usando una configuración basada en el entorno.

Simplemente crea una carpeta con un nombre que sea igual al de tu entorno de desarrollo dentro del directorio `config`, por ejemplo `local`. Después, crea los archivos de configuración que desees sobreescribir y especifica las opciones para tu entorno. Por ejemplo, para sobreescribir la configuración de caché para el entorno local, crearías el archivo `cache.php` en el directorio `app/config/local` con el siguiente contenido:

	<?php

	return array(

		'driver' => 'file',

	);

> **Nota:** No uses 'testing' como nombre para tu entorno. Éste está reservado para pruebas unitarias.

Observa que no necesitas especificar _cada_ opción disponible en el archivo de configuración predeterminado, sólo aquellas opciones que desees sobreescribir. Los archivos de configuración de tu entorno se aplicarán en "cascada" sobre los archivos predeterminados.

A continuación, necesitamos decirle al framework como determinar que entorno se está ejecutando. El entorno por defecto es siempre `production`. Sin embargo, puedes configurar otros entornos dentro del archivo `boostrap/start.php` en la raíz de tu instalación. En este archivo te encontrarás con un llamado a `$app->detectEnvironment`. El arreglo pasado a este método es usado para determinar el entorno de ejecución actual. Puedes agregar otros entornos y nombres de máquinas al arreglo como necesites.

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

Puedes también pasar argumentos al método `environment` para comprobar si el entorno concuerda con el valor dado:

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

Al usar configuración por entorno, podrías querer "añadir" [proveedores de servicio](/page/ioc#service-providers) de entorno a tu archivo de configuración `app` principal. Sin embargo, si pruebas esto, te darás cuenta que los proveedores de tu entorno están sobreescribiendo los proveedores en tu archivo de configuración `app` principal. Para forzar a los proveedores a ser añadidos, usa el método de ayuda `append_config` en tu archivo de configuración de entorno `app`.

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## Protegiendo configuración sensible

Para aplicaciones 'reales', es aconsejable mantener la configuración delicada fuera de los archivos de configuración. Cosas como contraseñas de bases de datos, llaves de la API de Stripe, y llaves de encriptación deberían permanecer fuera de los archivos de configuración siempre que sea posible. Así qué, ¿Dónde deberían guardarse? Afortunadamente, Laravel provee una solución muy simple para proteger ese tipo de configuración usando archivos "punto".<!-- TODO "dot" -->

Primero, [configura tu aplicación](/page/configuration#environment-configuration) para hacer reconocible tu máquina en el entorno `local`. Después, crea el archivo `.env.local.php` en la raíz de tu proyecto, que es usualmente el mismo directorio que contiene el archivo `composer.json`. El archivo `.env.local.php` debe retornar un arreglo de parejas llave-valor, tal como un archivo típico de configuración de Laravel:

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

Todos las parejas llave-valor retornadas por este archivo están automáticamente disponibles a través de las variables "superglobales" `$_ENV` y `$_SERVER` de PHP. Ahora puedes hacer referencia a estas variables globales en los archivos de configuración:

	'key' => $_ENV['TEST_STRIPE_KEY']

Asegúrate agregar el archivo `.env.local.php` a tu archivo `.gitignore`. Esto permitirá a otros desarrolladores en tu equipo crear su propia configuración de entorno local, así como ocultar configuración sensible del software de control de versiones.

Ahora, en tu servidor de producción, crea el archivo `.env.php` en el directorio raíz de tu proyecto, este guardará la configuración para tu entorno de producción. Así como el archivo `.env.local.php`, el archivo de producción `.env.php` no debería incluirse en el software de control de versiones.

> **Nota:** Puedes crear un archivo para cada entorno soportado por tu aplicación. Por ejemplo, el entorno `development` cargará el archivo `.env.development.php` si existe.

<a name="maintenance-mode"></a>
## Modo mantenimiento

Cuando tu aplicación está en modo mantenimiento, una vista personalizada se mostrará para todas las rutas de tu aplicación, esto hace que sea muy fácil "deshabilitar" tu aplicación mientras se está actualizando o cuando estás ejecutando un mantenimiento. Un llamado al método `App::down` ya se encuentra presente en tu archivo `app/start/global.php`. La respuesta de este método será enviada a los usuarios cuando tu aplicación está en modo mantenimiento.

Para habilitar el modo mantenimiento, simplemente ejecuta el comando de Artisan `down`:

	php artisan down

Para deshabilitar el modo mantenimiento, usa el comando `up`:

	php artisan up

Para mostrar una vista personalizada cuando tu aplicación este en modo mantenimiento, puedes agregar algo similar a esto al archivo `app/start/global.php` de tu aplicación:

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});

Si la Clausura pasada al método `down` retorna `NULL`, el modo mantenimiento será ignorado para esa petición.

### Modo mantenimiento y colas de trabajo

Mientras tu aplicación este en modo mantenimiento, no se ejecutarán [trabajos en cola](/page/queues). Los trabajos serán ejecutados normalmente una vez la aplicación ya no este en modo mantenimiento.