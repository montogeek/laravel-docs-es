# Desarrollo con Artisan

- [Introdución](#introduction)
- [Construir un Comando](#building-a-command)
- [Registrar Comandos](#registering-commands)
- [Llamar a Otros Comandos](#calling-other-commands)

<a name="introduction"></a>
## Introdución

Ademas de los comandos provistos con Artisan, puedes tambien construir tus propios comandos personalizados para trabajar con tu aplicación. Debes almacenar tus comandos personalizados en el directorio `app/commands`; Sin embargo, eres libre de seleccionar tu propia ubicación de almacenamiento siempre que tus comandos puedan ser autocargados basado en las configuraciones de tu `composer.json`.

<a name="building-a-command"></a>
## Construir un Comando

### Generar la Clase

Para generar un nuevo comando, debes uar el comando de Artisan `command:make`, el cual generará un trozo de código del comando para ayudarte a empezar:

#### Generar una Nueva Clase de Comando

	php artisan command:make FooCommand

Por defecto, los comandos generados serán alamacenados en el direcrorio `app/commands`; No obstante, puedes especificar la ruta o espacio de nombres:

	php artisan command:make FooCommand --path=app/classes --namespace=Classes

Cuando se está creando el comando, la opción `--command` puede ser usada para asignar el nombre del comando en la consola:

	php artisan command:make AssignUsers --command=users:assign

### Escribir el Comando

Ua vez tu comando es generado, debes llenar las propiedades `name` y `description` de la clase, la cuales serán usadas cuando se muestre tu comando en la pantalla `list`.

El método `fire` se llamará cuando tu comando es ejeutado. Debes poner cualquier logica del comando en este método.

### Argumentos & Opciones

Los métodos `getArguments` y `getOptions` están donde definas argumentos u opciones que tu comando reciba. Ambos métodos retornan un arreglo de comandos, los cuales son descritos por una lista de opciones de arreglo.

Al definir `argumentos`, la definición de valores del arreglo representa lo siguiente:

	array($name, $mode, $description, $defaultValue)

El argumento `mode` puede ser cualquiera de los siguientes: `InputArgument::REQUIRED` or `InputArgument::OPTIONAL`.

Al definir `opciones`, la definición de valores del arreglo representa lo siguiente:

	array($name, $shortcut, $mode, $description, $defaultValue)

Para opciones, el argumento `mode` puede ser: `InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE`.

El modo `VALUE_IS_ARRAY` indica que la opción puede usarse multiples veces cuando se llame el comando:

	php artisan foo --option=bar --option=baz

La opción `VALUE_NONE` indica que la opción es usada simplemente como un interruptor "switch":

	php artisan foo --option

### Obtener una Entrada "Input"

Mientras tu comando se está ejecutando, obviamente necesitaras acceder a los valores para los argumentos y opciones aceptados por tu aplicación. Para hacerlo,puedes usar los métodos `argument` y `option`:

#### Obtener el Valor de un Argumento en un Comando

	$value = $this->argument('name');

#### Obtener Todos los Argumentos

	$arguments = $this->argument();

#### Obtener el Valor de una Opción en un Comando

	$value = $this->option('name');

#### Obtener Todas las Opciones

	$options = $this->option();

### Escribir una Salida "Output"

Para enviar una salida a la consola, puedes usar los métodos `info`, `comment`, `question` y `error`. Cada uno de estos métodos usa los colores ANSI apropiados para su propósito.

#### Enviar Información a la Consola

	$this->info('Display this on the screen');

#### Enviar un Mensaje de Error a la Consola

	$this->error('Something went wrong!');

### Hacer Preguntas

También puedes usar los métodos `ask` y `confirm` para solicitarle una entrada al usuario:

#### Solicitar una Entrada al Usuario

	$name = $this->ask('What is your name?');

#### Solicitar una Entrada Secreta al Usuario

	$password = $this->secret('What is the password?');

#### Solicitar una Confirmación al Usuario

	if ($this->confirm('Do you wish to continue? [yes|no]'))
	{
		//
	}

También puedes especificar un valor por defecto para el método `confirm`, el cual que debe ser `true` or `false`:

	$this->confirm($question, true);

<a name="registering-commands"></a>
## Registrar Comandos

Una vez tu comando esté finalizado, necesitas registrarlo con Artisan para que esté disponible para su uso. Esto se hace tipicamente en el archivo `app/start/artisan.php`. Dentro de este archivo, puedes usar el método `Artisan::add` para registrar el comando:

#### Registrar un Comando de Artisan

	Artisan::add(new CustomCommand);

Si tu comando está registrado en la aplicación [IoC container](/4.1/ioc), puedes usar el método `Artisan::resolve` para que esté disponibla en Artisan:

#### Registrar un Comando que está en el IoC Container

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## LLamar a Otros Comandos

Algunas veces es posbile que desees llamar a otros comandos desde tu comando. Puedes hacerlo usando el método `call`:

#### Llamar a Otro Comando

	$this->call('command:name', array('argument' => 'foo', '--option' => 'bar'));
