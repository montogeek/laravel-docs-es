# Uso básico de la base de datos

- [Configuración](#configuration)
- [Leer / Escribir Conexiones](#read-write-connections)
- [Ejecutar consultas](#running-queries)
- [Transacciones de base de datos](#database-transactions)
- [Acceder conexiones](#accessing-connections)
- [Registro de consultas](#query-logging)

<a name="configuration"></a>
## Configuración

Laravel hace que la conexión a base de datos y la ejecución de sentencias extremadamente fácil. El archivo de configuración de base de datos es `app/config/database.php`. En este archivo puedes definir todas tus conexiones de base de datos, así como cual conexión se usará de forma predeterminada. En este archivo hay ejemplos de todos los motores de bases de datos soportados por el framework.

Actualmente Laravel soporta los sistemas de base de datos: MySQL, Postgres, SQLite y SQL Server.

<a name="read-write-connections"></a>
## Conexiones de Lectura / Escritura

Algunas puedes necesitar una conexión de base de datos para sentencias SELECT y otra para sentencias INSERT, UPDATE y DELETE. Laravel hace esto extremadamente fácil usando las conexiones apropiadas, no importa si estás ejecutando sentencias planas, el constructor de consultas o el ORM Eloquent.

Mira el siguiente ejemplo para saber como se deben configurar las conexiones de lectura y escritura:

	'mysql' => array(
		'read' => array(
			'host' => '192.168.1.1',
		),
		'write' => array(
			'host' => '196.168.1.2'
		),
		'driver'    => 'mysql',
		'database'  => 'database',
		'username'  => 'root',
		'password'  => '',
		'charset'   => 'utf8',
		'collation' => 'utf8_unicode_ci',
		'prefix'    => '',
	),

Tenga en cuenta que se han agregado dos índices al arreglo de configuración: `read` y `write`. Los dos índices tienen un arreglo como valor de un solo índice: `host`. El resto de las opciones de configuración de la base de datos para las conexiones `read` y `write` se combinarán desde el arreglo principal `mysql`. Así, solo necesitamos definir los valores de los arreglos `read` y `write` si deseamos sobreescribir los del arreglo principal. En este caso `192.168.1.1` será usada como la conexión de "lectura", mientras que `192.168.1.2` será usada como la conexión de escritura. El acceso a la base de datos, el prefijo, el conjunto de caracteres y las otras opciones en el arreglo principal `mysql` serán compartidas en las dos conexiones.

<a name="running-queries"></a>
## Ejecutar consultas

Una vez tengas configuradas las conexiones a las bases de datos, puedes ejecutar consultas usando la clase `BD`.

#### Ejecutando una sentencia SELECT

	$results = DB::select('select * from users where id = ?', array(1));

El método `select` siempre retornará un `array` de resultados.

#### Ejecutando una sentencia INSERT

	DB::insert('insert into users (id, name) values (?, ?)', array(1, 'Dayle'));

#### Ejecutando una sentencia UPDATE

	DB::update('update users set votes = 100 where name = ?', array('John'));

#### Ejecutando una sentencia DELETE

	DB::delete('delete from users');

> **Nota:** Los métodos `update` y `delete` retornan el número de filas afectadas por la sentencia.

#### Ejecutando una sentencia

	DB::statement('drop table users');

Puedes escuchar eventos de consultas usando el método `DB::listen`

#### Escuchar eventos de consulta

	DB::listen(function($sql, $bindings, $time)
	{
		//
	});

<a name="database-transactions"></a>
## Transacciones de base de datos

Para ejecutar un conjunto de sentencias dentro de una transacción, puedes usar el método `transaction`:

	DB::transaction(function()
	{
		DB::table('users')->update(array('votes' => 1));

		DB::table('posts')->delete();
	});

> **Nota:** Any exception thrown within the `transaction` closure will cause the transaction to be rolled back automatically.

Algunas veces puedes necesitar inicializar la transacción por ti mismo:

	DB::beginTransaction();

Puedes deshacer una transacción con el método `rollback`:

	DB::rollback();

Por último, puedes confirmar una transacción con el método `commit`:

	DB::commit();

<a name="accessing-connections"></a>
## Acceder conexiones

Cuando usas varias conexciones, puedes acceder a ellas a través del método `DB::connection`:

	$users = DB::connection('foo')->select(...);

Puedes también acceder a la simple y subyacente instancia PDO:

	$pdo = DB::connection()->getPdo();

Algunas veces puedes necesitar reconectarte a la base de datos:

	DB::reconnect('foo');

Si necesitas desconectarte de una base de datos debido a que se excede el límite `max_connections` de la instancia PDO subyacente, usa el método `disconnect`:

	DB::disconnect('foo');

<a name="query-logging"></a>
## Registro de consultas

De forma predeterminada, Laravel mantiene un registro en memoria de todas las consultras que se han ejecutado para le petición actual. Sin embargo, en algunos casos, como insertar una gran cantidad de filas puede causar que la aplicación excede la memoria. Para deshabilitar el registro, puedes usar el método `disableQueryLog`:

	DB::connection()->disableQueryLog();

Para obtener un arreglo de las consultar ejecutadas, puedes usar el método `getQueryLog`:

       $queries = DB::getQueryLog();
