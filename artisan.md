# Artisan CLI

- [Introducción](#introduction)
- [Uso](#usage)
- [Ejecutar comandos externos a CLI](#calling-commands-outside-of-cli)
- [Programar comandos de Artisan](#scheduling-artisan-commands)

<a name="introduction"></a>
## Introducción
Artisan es el nombre de la interfaz de línea de comando incluída en Laravel. Este brinda varios comandos útiles en el desarrollo de su aplicación. Este es impulsado por el poderoso componente Symfony Console.

<a name="usage"></a>
## Uso

#### Listar todos los comandos disponibles
Para ver una lista de todos los comandos disponibles en Artisan usted puede utilizar el comando `list`:

	php artisan list

#### Ver la ayuda para un comando

Cada comando incluye una pantalla de ayuda la cual despliega y describe los argumentos disponibles para el comando y sus opciones. Para ver una pantalla de ayuda simplemente anteponga `help` al nombre del comando:

	php artisan help migrate

#### Especificar el entorno de configuración

Usted puede especificar la configuración del ambiente que debe ser utilizada mientras ejecuta un comando usando el argumento `--env`:

	php artisan migrate --env=local

#### Mostrar la versión actual de Laravel
Usted puede ver la versión de su instalación de Laravel utilizando la opción `--version`:

	php artisan --version

<a name="calling-commands-outside-of-cli"></a>
## Ejecutar comandos externos a CLI
Algunas veces usted puede desear ejecutar un comando de Artisan externamente a la línea de comando. Por ejemplo usted puede querer ejecutar un comando de Artisan desde una ruta HTTP. Simplemente utilice el "facade" `Artisan`:

	Route::get('/foo', function()
	{
		$exitCode = Artisan::call('command:name', ['--option' => 'foo']);

		//
	});

Incluso puede poner los comandos Artisan en una cola para que sean procesados en segundo plano por su [queue workers](/5.0/queues):

	Route::get('/foo', function()
	{
		Artisan::queue('command:name', ['--option' => 'foo']);

		//
	});

<a name="scheduling-artisan-commands"></a>
## Programar comandos de Artisan

In the past, developers have generated a Cron entry for each console command they wished to schedule. However, this is a headache. Your console schedule is no longer in source control, and you must SSH into your server to add the Cron entries. Let's make our lives easier. The Laravel command scheduler allows you to fluently and expressively define your command schedule within Laravel itself, and only a single Cron entry is needed on your server.

Your command schedule is stored in the `app/Console/Kernel.php` file. Within this class you will see a `schedule` method. To help you get started, a simple example is included with the method. You are free to add as many scheduled jobs as you wish to the `Schedule` object. The only Cron entry you need to add to your server is this:

	* * * * * php /path/to/artisan schedule:run 1>> /dev/null 2>&1

This Cron will call the Laravel command scheduler every minute. Then, Laravel evaluates your scheduled jobs and runs the jobs that are due. It couldn't be easier!

### Más ejemplos de programación de comandos

Let's look at a few more scheduling examples:

#### Clausura de programación de comandos

	$schedule->call(function()
	{
		// Do some task...

	})->hourly();

#### Programando comandos de terminal

	$schedule->exec('composer self-update')->daily();

#### Expresión CRON manual

	$schedule->command('foo')->cron('* * * * *');

#### Frecuencia de Jobs

	$schedule->command('foo')->everyFiveMinutes();

	$schedule->command('foo')->everyTenMinutes();

	$schedule->command('foo')->everyThirtyMinutes();

#### Jobs diarios

	$schedule->command('foo')->daily();

#### Jobs diarios un un momento especifico (Formato 24 horas)

	$schedule->command('foo')->dailyAt('15:00');

#### Jobs dos veces por dia

	$schedule->command('foo')->twiceDaily();

#### Job cada fin de semana

	$schedule->command('foo')->weekdays();

#### Jobs semanales

	$schedule->command('foo')->weekly();

	// Schedule weekly job for specific day (0-6) and time...
	$schedule->command('foo')->weeklyOn(1, '8:00');

#### Jobs mensuales

	$schedule->command('foo')->monthly();

#### Job ejecutados en un día específico

	$schedule->command('foo')->mondays();
	$schedule->command('foo')->tuesdays();
	$schedule->command('foo')->wednesdays();
	$schedule->command('foo')->thursdays();
	$schedule->command('foo')->fridays();
	$schedule->command('foo')->saturdays();
	$schedule->command('foo')->sundays();

#### Prevenir Jobs se sobreescriban

De forma predeterminada, jobs programados se ejecutaran incluso si la instancia previa del job aun esta coriendo. Para prevenir esto, puedes usar el metodo `withoutOverLapping`:

	$schedule->command('foo')->withoutOverLapping();

En este ejemplo, el comando `foo` se ejecutara cada minuto si ya no esta corriendo.

#### Job ejecutado en un entorno específico

	$schedule->command('foo')->monthly()->environments('production');

#### Ejecutar el Job incluso con la aplicación en mantenimiento

	$schedule->command('foo')->monthly()->evenInMaintenanceMode();

#### Solo ejecutar un Job cuando el callback retorna `true`

	$schedule->command('foo')->monthly()->when(function()
	{
		return true;
	});

#### Enviar la salida del Job a un E-mail

	$schedule->command('foo')->sendOutputTo($filePath)->emailOutputTo('foo@example.com');

> **Note:** You must send the output to a file before it can be mailed.

#### Guardar la salida del Job a una locación en específica

	$schedule->command('foo')->sendOutputTo($filePath);

#### Ping una URL cuando el Job se ejecute

	$schedule->command('foo')->thenPing($url);
