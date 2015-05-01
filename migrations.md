# Migraciones y Semillas

- [Introducción](#introduction)
- [Creando migraciones](#creating-migrations)
- [Ejecutando migraciones](#running-migrations)
- [Deshacer migraciones](#rolling-back-migrations)
- [Poblado de la base de datos](#database-seeding)

<a name="introduction"></a>
## Introducción

Las migraciones son un tipo de control de versiones para tu base de datos. Permiten que un equipo modifique la estructura de la base de datos y permanecer actualizados con el estado de la estructura actual. Las migraciones están enlazadas habitualmente con el [Constructor de esquemas](/5.0/schema) para gestionar con facilidad el esquema de tu aplicación.

<a name="creating-migrations"></a>
## Creando Migraciones

Para crear una migración, puedes usar el comando `make:migration` de la línea de comandos de Artisan:

	php artisan make:migration create_users_table

La migración será colocada en tu directorio `database/migrations`, y contendrá una marca de tiempo que permitirá al framework determinar el orden de las migraciones.

Cuando creas una migración, también puedes especificar la opción `--path`. La ruta debería ser relativa al directorio raíz de tu instalación: <sup>[1](#nota1)</sup>

	php artisan make:migration foo --path=app/migrations

También puedes usar las opciones `--table` y `--create` para indicar el nombre de la tabla, y si la migración creará una nueva tabla:

	php artisan make:migration add_votes_to_user_table --table=users

	php artisan make:migration create_users_table --create=users

<a name="running-migrations"></a>
## Ejecutando Migraciones

#### Ejecutando Todas las Migraciones Pendientes

	php artisan migrate

> **Nota:** Si recibes un error "class not found" ('clase no encontrada'), inténtalo ejecutando el comando `composer dump-autoload`.

### Forzar las Migraciones en Producción.

Algunas operaciones de migración son destructivas, es decir, pueden causar pérdida de datos. Para protegerte de ejecutar esos comandos contra tu base de datos de producción, se te pedirá confirmación antes de ejecutar esos comandos. Para forzar que los comandos se ejecuten sin preguntar, usa el flag `--force`:

	php artisan migrate --force

<a name="rolling-back-migrations"></a>
## Deshacer Migraciones

#### Deshacer la Última Operación de Migración (Rollback)

	php artisan migrate:rollback

#### Deshacer todas las migraciones

	php artisan migrate:reset

#### Deshacer todas las migraciones y ejecutándolas otra vez.

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## Poblado de la base de datos

Laravel incluye también un sencillo método para llenar tus bases de datos con datos de prueba usando clases 'seed'. Todas las clases para el poblado de datos son almacenadas en `database/seeds`. Estas clases pueden tener cualquier nombre que quieras. Pero, probablemente, deberían seguir algunas convenciones como `UserTableSeeder` (`PobladorDeTablaUsuarios`), etc.. Por defecto, se define una clase `DatabaseSeeder` por ti. Desde esta clase, puedes usar el método `call` para ejecutar otras clases de poblado de datos. Permitiéndote controlar el orden en el que se produce ese poblado de datos.

#### Ejemplo de Clase de Poblado de Datos

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('UserTableSeeder');

			$this->command->info('User table seeded!');
		}

	}

	class UserTableSeeder extends Seeder {

		public function run()
		{
			DB::table('users')->delete();

			User::create(array('email' => 'foo@bar.com'));
		}

	}

Para llenar de datos tu base de datos, puedes usar el comando `db:seed` de la línea de comandos de Artisan:

	php artisan db:seed

Por defecto, el comando `db:seed` ejecuta la clase `DatabaseSeeder`, la cual puede ser usada para llamar a otras clases de poblado de datos. Sin embargo, puedes usar la opción `--class` para especificar que una clase de poblado de datos concreta se ejecute individualmente:

	php artisan db:seed --class=UserTableSeeder

También puedes poblar tus bases de datos usando el comando `migrate:refresh`, que deshace (rollback) y vuelve a ejecutar todas tus migraciones:

	php artisan migrate:refresh --seed

<br />
** Notas de la traducción:

<a name="nota1">[1]</a> `foo` es un término que se usa habitualmente en inglés para referirse a cualquier variable o representación informática con un nombre genérico que podría traducirse por 'cualquier nombre', 'cualquier texto', etc... Por sí misma, la palabra foo no tiene un significado preciso; Se usa como nombre genérico. En castellano sería como decir 'fulanito' o 'menganito'.
