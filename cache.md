# Caché

- [Configuración](#configuration)
- [Uso de Caché](#cache-usage)
- [Incrementos & Decrementos](#increments-and-decrements)
- [Etiquetas de Caché](#cache-tags)
- [Caché en Base de Datos](#database-cache)

<a name="configuration"></a>
## Configuración

Laravel provee una API unificada para vario sistemas de almacenamiento en caché. La configuración de la caché se encuentra en `app/config/cache.php`. En este archivo puedes especificar cuál controlador de caché te gustaría usar por defecto en a lo largo de tu aplicación. Laravel soporta backends de almacenamiento en caché populares como [Memcached](http://memcached.org) and [Redis](http://redis.io) los cuales funcionan inmediatamente después de la instalación sin ninguna configuración o modificación ("out of the box").

El archivo de configuración de la caché además contiene otras opciones, las cuales estan documentadas dentro del archivo mismo, así que asegurate de leer más de estas opciones. Por defecto, Laravel está configurado para usar el controlador de caché `file`, el cual almacena los objetos serealizados en cache, en el sistema de archivos. Para grandes aplicaciones, se recomienda que uses caché en memoria como Memcached o APC.

<a name="cache-usage"></a>
## Uso de Caché

#### Almacenar un Elmento en la Caché

	Cache::put('key', 'value', $minutes);

#### Usar Objetos Carbon para Establecer el Tiempo de Caducidad

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

#### Almacenar un Elemento en la Caché sí este no Existe

	Cache::add('key', 'value', $minutes);

El método `add` retornará `true` si el elemento efectivamente fué **añadido** a la caché. Si no, el método retornará `false`.

#### Comprobar la Existencia de un Elemento en Caché

	if (Cache::has('key'))
	{
		//
	}

#### Obtener un Elemento de la Caché

	$value = Caché::get('key');

#### Obtener un Elemento o Retornar un Valor por Defecto

	$value = Caché::get('key', 'default');

	$value = Caché::get('key', function() { return 'default'; });

#### Almacenar un Elemento Permanentemente en la Caché

	Cache::forever('key', 'value');

Algunas veces es posible que desees obtener un elemento de la caché, pero también almacenar un valor por defecto cuando el elemento solicitado no existe. Puedes hacer esto usando el método `Cache::remember`:

	$value = Caché::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

También puedes combinar los métodos `remember` y `forever`:

	$value = Caché::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

Nota que todos los elementos almacenados en la chacé están serializados, asi que eres libre de almacenar cualquier tipo de dato.

#### Eliminar un Elemento de la Caché

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## Incrementos & Decrementos

Todos los controladores exceptuando `file` y `database` soportan las operaciones `increment` y `decrement`:

#### Incrementar un Valor

	Cache::increment('key');

	Cache::increment('key', $amount);

#### Decrementar un Valor

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-tags"></a>
## Etiquetas de Caché

> **Nota:** Las etiquetas Caché no son soportadas cuando se usan los controladores de caché `file` o `database`. Es más, al usar varias etiquetas con cachés que están almacenados permanentemente "forever", el desempeño será mejor con un controlador como `memcached`, el cual atomáticamente depura los registros obsoletos.

Las etiquetas Caché te permiten etiquetar elementos relacionados en la caché, y luego vaciar todas las cachés etiquetadas con un nombre dado. Para acceder  una caché etiquetada, usa el método `tags`:

#### Acceder una Caché Etiquetada

Puedes almacenar una caché etiquetada pasando como argumentos una lista ordenada o un arreglo ordenado de nombres de etiquetas.

	Cache::tags('people', 'authors')->put('John', $john, $minutes);

	Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);

Puedes usar cualquier método de almacenamiento de caché en combinación con etiquetas, incluyendo `remember`, `forever`, y `rememberForever`. También puedes accesar los elementos cacheados de la caché etiquetada, así como usar otros métodos de caché como lo son `increment` y `decrement`:

#### Acceder a Elementos en una Caché Etiquetada

Para acceder una caché etiquetada, pasa la misma lista ordenada de etiquetas usada para guardarla.

	$anne = Caché::tags('people', 'artists')->get('Anne');

	$john = Caché::tags(array('people', 'authors'))->get('John');


Puedes vaciar todos los elementos etiquetados con un nombre o lista de nombres. Por ejemplo, esta sentencia eliminará todas las cachés etiquetadas ya sea con `people`, `authors`, o ambos. Asi que, ambos "Anne" y "John" serán eliminados de la caché:
	Cache::tags('people', 'authors')->flush();

Por el contrario, esta sentencia elimina unicamente las cachés etiquetadas con `authors`, asi que "John" será eliminado, pero "Anne" no.

	Cache::tags('authors')->flush();

<a name="database-cache"></a>
## Caché en Base de Datos

Cuando se está usando el controlador de caché `database`, necesitarás configurar una tabla que contenga los elementos de caché. Debajo encontrarás un ejemplo de declaración `Schema` para la tabla:

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});
