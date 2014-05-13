# Constructor de consultas

- [Introducción](#introduction)
- [Selects](#selects)
- [Joins](#joins)
- [Wheres avanzados](#advanced-wheres)
- [Agregados](#aggregates)
- [Expresiones planas](#raw-expressions)
- [Inserts](#inserts)
- [Updates](#updates)
- [Deletes](#deletes)
- [Unions](#unions)
- [Bloqueo pesimista](#pessimistic-locking)
- [Cache de consultas](#caching-queries)

<a name="introduction"></a>
## Introducción

El constructor de consulta de base de datos ofrece una cómoda, conveniente y fluída forma para crear y ejecutar consultas de base de datos. Puede ser usado para ejecutar la mayoría de las operaciones con la base de datos en tu aplicación, y funciona en todos los motores de base de datos soportados.

> **Nota:** El constructor de consultas de Laravel usa vinculación de parámetros PDO en todas las consultas para proteger tu aplicación de ataques de inyección de SQL. No hay necesidad de limpiar las cadenas de texto que son pasadas como vínculos.

<a name="selects"></a>
## Selects

#### Obtener todas las filas de una tabla

	$users = DB::table('users')->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### Obtener una sola fila de una tabla

	$user = DB::table('users')->where('name', 'John')->first();

	var_dump($user->name);

#### Obtener una sola columna de una fila

	$name = DB::table('users')->where('name', 'John')->pluck('name');

#### Obtener una lista de valores de una columna

	$roles = DB::table('roles')->lists('title');

Éste método retornará una rreglo de títulos de roles. Puedes también especificar un índice de una columna personalizada para el arreglo retornado:

	$roles = DB::table('roles')->lists('title', 'name');

#### Especificar una sentencia Select

	$users = DB::table('users')->select('name', 'email')->get();

	$users = DB::table('users')->distinct()->get();

	$users = DB::table('users')->select('name as user_name')->get();

#### Agregar una sentencia Select a una consulta previa

	$query = DB::table('users')->select('name');

	$users = $query->addSelect('age')->get();

#### Usar operadores Where

	$users = DB::table('users')->where('votes', '>', 100)->get();

#### O sentencias

	$users = DB::table('users')
	                    ->where('votes', '>', 100)
	                    ->orWhere('name', 'John')
	                    ->get();

#### Usar Where entre

	$users = DB::table('users')
	                    ->whereBetween('votes', array(1, 100))->get();

#### Usar Where no entre

	$users = DB::table('users')
	                    ->whereNotBetween('votes', array(1, 100))->get();

#### Usar Where In en un array

	$users = DB::table('users')
	                    ->whereIn('id', array(1, 2, 3))->get();

	$users = DB::table('users')
	                    ->whereNotIn('id', array(1, 2, 3))->get();

#### Usar Where Null para encontrar registros con valores no definidos

	$users = DB::table('users')
	                    ->whereNull('updated_at')->get();

#### Usar Order By, Group By, And Having

	$users = DB::table('users')
	                    ->orderBy('name', 'desc')
	                    ->groupBy('count')
	                    ->having('count', '>', 100)
	                    ->get();

#### Usar Offset y Limit

	$users = DB::table('users')->skip(10)->take(5)->get();

<a name="joins"></a>
## Uniones

El constructor de consultras puede además ser usado para escribir sentencias JOIN. Mira los siguientes ejemplos:

#### Sentencia básica de Join

	DB::table('users')
	            ->join('contacts', 'users.id', '=', 'contacts.user_id')
	            ->join('orders', 'users.id', '=', 'orders.user_id')
	            ->select('users.id', 'contacts.phone', 'orders.price')
	            ->get();

#### Sentencia Left Join

	DB::table('users')
		    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
		    ->get();

Puedes también crear sentencias JOIN más avanzadas:

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')->orOn(...);
	        })
	        ->get();

Si deseas usar una sentencia estilo "where" en tus sentencias JOIN, puedes usar el método `where` y `orWhere` en el método `join`. En vez de comparar dos columnas, estos métodos compararán la columna contra un valor:

	DB::table('users')
	        ->join('contacts', function($join)
	        {
	        	$join->on('users.id', '=', 'contacts.user_id')
	        	     ->where('contacts.user_id', '>', 5);
	        })
	        ->get();

<a name="advanced-wheres"></a>
## Wheres Avanzados

Algunas veces puedes necesitar la ccreación de sentencias where más avanzadas como "where exists" o agrupar jerarquicamente parámetros. El constructor de consultas de Laravel pueden manejar esto también:

#### Agrupación de parámetros

	DB::table('users')
	            ->where('name', '=', 'John')
	            ->orWhere(function($query)
	            {
	            	$query->where('votes', '>', 100)
	            	      ->where('title', '<>', 'Admin');
	            })
	            ->get();

La consulta anterior generará la siguiente consulta SQL:

	select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

#### Sentencias Exists

	DB::table('users')
	            ->whereExists(function($query)
	            {
	            	$query->select(DB::raw(1))
	            	      ->from('orders')
	            	      ->whereRaw('orders.user_id = users.id');
	            })
	            ->get();

La consulta anterior generará la siguiente consulta SQL:

	select * from users
	where exists (
		select 1 from orders where orders.user_id = users.id
	)

<a name="aggregates"></a>
## Agregados

En constructor de consultas además ofrece una variedad de métodos de agregación, como `count`, `max`, `min`, `avg` y `sum`.

#### Usando los métodos

	$users = DB::table('users')->count();

	$price = DB::table('orders')->max('price');

	$price = DB::table('orders')->min('price');

	$price = DB::table('orders')->avg('price');

	$total = DB::table('users')->sum('votes');

<a name="raw-expressions"></a>
## Expresiones planas

Algunas veces puedes necesitar el uso de expresiones planas en una consulta. Dichas expresiones serán inyectadas dentro de la consulta como cadenas de texto, así que ten cuidado de no crear algún punto débil para inyecciones SQL! Para crear una expresión plana, puedes usar el método `DB::raw`:

#### Usar una expresión plana

	$users = DB::table('users')
	                     ->select(DB::raw('count(*) as user_count, status'))
	                     ->where('status', '<>', 1)
	                     ->groupBy('status')
	                     ->get();

#### Aumentar o disminuir el valor de una columna:

	DB::table('users')->increment('votes');

	DB::table('users')->increment('votes', 5);

	DB::table('users')->decrement('votes');

	DB::table('users')->decrement('votes', 5);

You may also specify additional columns to update:

	DB::table('users')->increment('votes', 1, array('name' => 'John'));

<a name="inserts"></a>
## Inserts

#### Insertando filas en una tabla

	DB::table('users')->insert(
		array('email' => 'john@example.com', 'votes' => 0)
	);

Si las tablas tienen un ID autoincrementable, usa el método `insertGetId` para insertar una fila y retornar el ID:

#### Insertar filas en una tabla con un ID autoincrementable

	$id = DB::table('users')->insertGetId(
		array('email' => 'john@example.com', 'votes' => 0)
	);

> **Nota:** Cuando estés usando PostgreSQL el método `insertGetId` espera que la columna autoincrementable se llame "id".

#### Insertar múltiples filas en una tabla

	DB::table('users')->insert(array(
		array('email' => 'taylor@example.com', 'votes' => 0),
		array('email' => 'dayle@example.com', 'votes' => 0),
	));

<a name="updates"></a>
## Updates

#### Actualizar filas en una tabla

	DB::table('users')
	            ->where('id', 1)
	            ->update(array('votes' => 1));

<a name="deletes"></a>
## Deletes

#### Eliminar filas en una tabla

	DB::table('users')->where('votes', '<', 100)->delete();

#### Eliminar todas las filas de una tabla

	DB::table('users')->delete();

#### Truncar una tabla

	DB::table('users')->truncate();

<a name="unions"></a>
## Unions

El constructor de consultas también ofrece una forma rápida para "unir" dos consultas en una sola:

#### Realizar la unión de dos consultas

	$first = DB::table('users')->whereNull('first_name');

	$users = DB::table('users')->whereNull('last_name')->union($first)->get();

El método `unionAll` está también disponible, y tiene la misma síntaxis que `union`.

<a name="pessimistic-locking"></a>
## Bloqueo pesimista

El constructor de consultas incluye algunas funciones que te ayudan a realizar "bloqueos pesimistas" en tus sentencias SELECT.

Para ejecutar una sentencia SELECT con un "bloqueo compartido", puedes usar el método `sharedLock` en la consulta:

	DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Para "bloquear para actualizaciones" en una sentencia SELECT, puedes usar el método `lockForUpdate` en la consulta:

	DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();

<a name="caching-queries"></a>
## Cache de consultas

Puedes fácilmente tener un cache de los resultados de una consulta con el método `remember`:

#### Crear un cache de los resultados de una consulta:

	$users = DB::table('users')->remember(10)->get();

En este ejemplo, los resultados de la consulta serán guardados en cache durante diez minutos. Mientras los resultados estén guardados en cache, la consulta no se ejecutará otra vez en la base de datos, y los resultados será cargados del cache predeterminado configurado en tu aplicación.

Si estás usando un [controlador de cache soportado](/docs/cache#cache-tags), puedes además agregar etiquetas al caché:

	$users = DB::table('users')->cacheTags(array('people', 'authors'))->remember(10)->get();
