# Laravel: Inicio rápido

- [Instalación](#instalacion)
- [Enrutado](#routing)
- [Creando una vista](#creating-a-view)
- [Creando una Migración](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [Mostrando informacion](#displaying-data)

<a name="installation"></a>
## Instalación

### Usando el instalador de Laravel

Primero, descarga el [archivo de instalación PHAR de laravel](http://laravel.com/laravel.phar). Por conveniencia, cambia el nombre del archivo a `laravel` y muevelo a la carpeta `/usr/local/bin`. Una vez instalado, el comando `laravel new` creará una instalación limpia de Laravel en el directorio especificado. Por ejemplo, `laravel new blog` crearía el directorio llamado `blog` que contendra una instalación limpia de Laravel con todas las dependencias instaladas. Este método de instalación es mas rápido que el instalar usando Composer.

### Usando Composer

El framework Laravel utiliza [Composer](http://getcomposer.org) para su instalación y la gestión de dependencias. Si no lo tiene instalado, puede empezar por [instalar Composer es su maquina](http://getcomposer.org/doc/00-intro.md).

Ahora puedes instalar Laravel utilizado el siguiente comando en tu consola:

	composer create-project laravel/laravel nombre-de-proyecto --prefer-dist

Este comando descargará e instalará una copia limpia de Laravel en la carpeta `nombre-de-proyecto` en directorio que se encuentre en el momento.

Si prefiere, otra alternativa es descargar manualmente una copia del [repositorio de Laravel en Github](https://github.com/laravel/laravel/archive/master.zip). Lo siguiente es ejecutar el comando `composer install` en el directorio raíz del proyecto creado manualmente con anterioridad. Este comando dscargará e instalará las dependencia del framework.

### Permisos

Despúes de instalar Laravel, s probablemente se necesite darle permisos de escritura al navegador para el directorio `app/storage`. Para más detalles de configuración ver la documentación de [Instalación](/4.1/installation).

### Ejecutar Laravel

Normalmente puedes usar un servidor Apache o Ngnix para ejecutar tus aplicaciones de Laravel. Si tienes una versión de PHP mayor a 5.4 puedes usar el servidor interno de Laravel, a través del comando de Artisan `serve`.

	php artisan serve

<a name="directories"></a>
### Estructura de los directorios

Después de instalar el framework, echale un vistazo al proyecto para familiarizarte con la estructura de los directorios. El directorio `app` contiene las carpetas `views`, `controllers`, y `models`. La mayoría del código de tu aplicación existirá en éste directorio. También puedes explorar la carpeta `app/config` y las opciones de configuración disponibles.

<a name="routing"></a>
## Rutas

Para empezar, creemos nuestra primer ruta. En Laravel, la ruta más simple es una Clausura. Abre el archivo `app/routes.php` y agrega la siguiente rua al final del archivo:

	Route::get('users', function()
	{
		return 'Users!';
	});

Ahora, si visitas la ruta `/users` en tu navegador, deberías ver `Users!` como respuesta. Genial! Has creado tu primer ruta..

Las rutas también pueden ser vinculadas a clases de Controladores. Por ejemplo:

	Route::get('users', 'UserController@getIndex');

Esta ruta se informa al framework que la ruta `/users` deberá ejecutar el método `getIndex` en la clase `UserController`. Para más información de rutas con controladores, visita la [documentación de controladores](/4.1/controllers).

<a name="creating-a-view"></a>
## Crear una vista

Ahora, crearemos una vista sencilla para mostrar la información de nuestro usuario. Las vistas se encuentran en la carpeta `app/views` y contienen el HTML de tu aplicación.
Vamos a crear dos nuevas vistas en esta carpeta `layout.blade.php` y `users.blade.php`. Primero, creemos nuestro archivo `layout.blade.php`: file:

	<html>
		<body>
			<h1>Laravel Quickstart</h1>

			@yield('content')
		</body>
	</html>

A continuación, nuestra vista `users.blade.php`

	@extends('layout')

	@section('content')
		Users!
	@stop

Alguna parte de la síntaxis probablemente sea extraña para ti. Es porque estamos usando el motor de plantillas de Laravel: Blade. Blade es muy rápido, porque simplemente son varias expresiones regulares ejecutadas a través de tus plantillas para ser compiladas en PHP nativo. Blade provee funcionalidad poderosa como herencia de plantillas, así como algunas mejoras de síntaxis en las estructuras de control de PHP como `if` y `for`. Revisa la documentación de [Blade](/4.1/templates) para más detalles.

Ahora que tenemos nuestras vistas, ejecutemoslas desde nuestra ruta `/users`. En vez de retornar `Users!` desde la ruta, retornaremos una vista:

	Route::get('users', function()
	{
		return View::make('users');
	});

Maravilloso! Has configurado una vista simple que extiende una plantilla. A continuación, empezemos con nuestra capa de base de datos.

<a name="creating-a-migration"></a>
## Crear un archivo de migración

To create a table to hold our data, we'll use the Laravel migration system. Migrations let you expressively define modifications to your database, and easily share them with the rest of your team.

First, let's configure a database connection. You may configure all of your database connections from the `app/config/database.php` file. By default, Laravel is configured to use MySQL, and you will need to supply connection credentials within the database configuration file. If you wish, you may change the `driver` option to `sqlite` and it will use the SQLite database included in the `app/database` directory.

Next, to create the migration, we'll use the [Artisan CLI](/4.1/artisan). From the root of your project, run the following from your terminal:

	php artisan migrate:make create_users_table

Next, find the generated migration file in the `app/database/migrations` folder. This file contains a class with two methods: `up` and `down`. In the `up` method, you should make the desired changes to your database tables, and in the `down` method you simply reverse them.

Let's define a migration that looks like this:

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

Next, we can run our migrations from our terminal using the `migrate` command. Simply execute this command from the root of your project:

	php artisan migrate

If you wish to rollback a migration, you may issue the `migrate:rollback` command. Now that we have a database table, let's start pulling some data!

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel ships with a superb ORM: Eloquent. If you have used the Ruby on Rails framework, you will find Eloquent familiar, as it follows the ActiveRecord ORM style of database interaction.

First, let's define a model. An Eloquent model can be used to query an associated database table, as well as represent a given row within that table. Don't worry, it will all make sense soon! Models are typically stored in the `app/models` directory. Let's define a `User.php` model in that directory like so:

	class User extends Eloquent {}

Note that we do not have to tell Eloquent which table to use. Eloquent has a variety of conventions, one of which is to use the plural form of the model name as the model's database table. Convenient!

Using your preferred database administration tool, insert a few rows into your `users` table, and we'll use Eloquent to retrieve them and pass them to our view.

Now let's modify our `/users` route to look like this:

	Route::get('users', function()
	{
		$users = User::all();

		return View::make('users')->with('users', $users);
	});

Let's walk through this route. First, the `all` method on the `User` model will retrieve all of the rows in the `users` table. Next, we're passing these records to the view via the `with` method. The `with` method accepts a key and a value, and is used to make a piece of data available to a view.

Awesome. Now we're ready to display the users in our view!

<a name="displaying-data"></a>
## Mostrando información

Now that we have made the `users` available to our view. We can display them like so:

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

You may be wondering where to find our `echo` statements. When using Blade, you may echo data by surrounding it with double curly braces. It's a cinch. Now, you should be able to hit the `/users` route and see the names of your users displayed in the response.

This is just the beginning. In this tutorial, you've seen the very basics of Laravel, but there are so many more exciting things to learn. Keep reading through the documentation and dig deeper into the powerful features available to you in [Eloquent](/4.1/eloquent) and [Blade](/4.1/templates). Or, maybe you're more interested in [Queues](/4.1/queues) and [Unit Testing](/4.1/testing). Then again, maybe you want to flex your architecture muscles with the [IoC Container](/4.1/ioc). The choice is yours!
