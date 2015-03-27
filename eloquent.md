# ORM Eloquent

- [Introducción](#introduction)
- [Uso básico](#basic-usage)
- [Asignación masiva](#mass-assignment)
- [Insertar, actualizar, borrar](#insert-update-delete)
- [Borrado blando](#soft-deleting)
- [Marcas de tiempo](#timestamps)
- [Consultas con ámbito](#query-scopes)
- [Consultas globales con ámbito](#global-scopes)
- [Relaciones](#relationships)
- [Consulta de relaciones](#querying-relations)
- [Carga impaciente](#eager-loading)
- [Inserta modelos asociados](#inserting-related-models)
- [Actualizar marcas de tiempo en relacioines padre](#touching-parent-timestamps)
- [Trabajar con tablas pivote](#working-with-pivot-tables)
- [Colecciones](#collections)
- [Accesores y mutadores](#accessors-and-mutators)
- [Mutadores de fecha](#date-mutators)
- [Forzar tipos](#attribute-casting)
- [Eventos de modelos](#model-events)
- [Observadores de modelos](#model-observers)
- [Generar URL para modelos](#model-url-generation)
- [Convertir a json / arreglos](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Introducción

El ORM Eloquent incluído con Laravel provee una implementación hermosa y sencilla de "ActiveRecord" para trabajar con su base de datos. Cada tabla de la base de datos tiene un "Modelo" correspondiente que es usado para interactuar con la tabla.

Antes de empezar, asegúrase de configurar la conexión a la base de datos en `config/database.php`.

<a name="basic-usage"></a>
## Uso básico

Para empezar, cree un modelo Eloquent. Generalmente los modelos están en el directorio `app`, pero siéntase en libertad de guardarlos en cualquier directorio que se pueda autocargar a través de su archivo `composer.json`. Todos los modelos Eloquent extienden `Illuminate\Database\Eloquent\Model`.

#### Definir un modelo Eloquent

	class User extends Model {}

También puede generar los modelos Eloquent usando el comando `make:model`:

	php artisan make:model User

Observe que no se dijo cual tabla se debe usar para el modelo `User`. El nombre de la clase en mínusculas se usará para el nombre de la tabla a menos que explícitamente se especifique uno diferente. Entonces, en este caso Eloquent asume que el modelo `User` almacena los registroes en la tabla `users`. Puede personalizar el nombre de la tabla con la propiedad `table` en su modelo.

	class User extends Model {

		protected $table = 'my_users';

	}

> **Nota:** Eloquent también asume que cada tabla tiene una columna `id` como llave primaria. Puede definir la propiedad `primaryKey` para cambiar esta convensión. Así mismo,  puede definir la propiedad `connection` para cambiar el nombre de la conexión a la base de datos que se debe usar cuando se use el modelo.

Una vez se define un modelo, ya está listo para comenzar la recuperación y la creación de registros en la tabla. Tenga en cuenta que tendrá que colocar las columnas `updated_at` y `created_at` en su tabla de forma predeterminada. Si no desea que estas columnas sean mantenidas automáticamente, establezca la propiedad `$timestamps` de su modelo en falso.

#### Obtener todos los modelos

	$users = User::all();

#### Obtener un registro por llave primaria

	$user = User::find(1);

	var_dump($user->name);

> **Nota:** Todos los métodos disponibles en el [generador de consultas](/docs/queries) también están disponibles al consultar modelos Elocuent.

#### Obtener un registro por llave primaria o generar una excepción

A veces es posible que desee lanzar una excepción si no se encuentra un registro, lo que le permite capturar las excepciones utilizando el manejador `App::error` y mostrar una página 404.

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

Para registrar el manejador de errores, escuche por `ModelNotFoundException`

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Not Found', 404);
	});

#### Consultas usando modelos Eloquent

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### Agregados Eloquent

Por supuesto, también puede utilizar las funciones de agregado del generador de consultas.

	$count = User::where('votes', '>', 100)->count();

Si usted es incapaz de generar la consulta que necesita a través de la interfaz fluida, no dude en utilizar `whereRaw`:

	$users = User::whereRaw('age > ? and votes = 100', [25])->get();

#### Resultado en trozos

Si usted necesita procesar una gran cantidad (miles) de registros Elocuent, usando el comando `chunk` permitirá que usted lo haga sin acabar toda su RAM:

	User::chunk(200, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});

El primer argumento pasado al método es el número de registros que desea recibir por "trozos". La Clausura (closure) pasada como segundo argumento se llamará para cada trozo que se extrae de la base de datos.

#### Especificar una conexión de consulta

También puede especificar qué conexión de base de datos se debe emplear cuando se ejecuta una consulta Elocuent. Sólo tiene que utilizar el método `on`:

	$user = User::on('connection-name')->find(1);

Si utiliza [conexiones lectura/escritura](/5.0/database#read-write-connections), es posible forzar la consulta para utilizar la conexión "escribir" con el siguiente método:

	$user = User::onWriteConnection()->find(1);

<a name="mass-assignment"></a>
## Asignación masiva 

Al crear un nuevo modelo, puede pasar un arreglo de atributos al constructor del modelo. Estos atributos se asignan al modelo a través de la asignación masiva. Esto es conveniente; Sin embargo, puede ser un **grave** problema de seguridad al pasar ciegamente la entrada del usuario en un modelo. Si la entrada del usuario se pasa a ciegas a un modelo, el usuario es libre de modificar **cualquier** o **todos** los atributos del modelo. Por esta razón, todos los modelos Elocuent por omisión se protegen contra la asignación masiva.

Para empezar, establezca la propiedad `fillable` o  `guarded` en su modelo.

#### Definir el atributo `fillable` en un modelo 

La propiedad `fillable` especifica qué atributos puen asignarse en masa. Esto se puede configurar a nivel de clase o instancia.

	class User extends Model {

		protected $fillable = ['first_name', 'last_name', 'email'];

	}

En este ejemplo, sólo los tres atributos enumerados serán asignables en masa.

#### Definir el atributo `guarded` en un modelo 

La propiedad inversa de `fillable` es `guarded`, y sirve como una "lista negra" en lugar de una "lista blanca":

	class User extends Model {

		protected $guarded = ['id', 'password'];

	}

> **Nota:** Al usar `guarded`, nunca debe pasar `Input::get()` o cualquier arreglo de entrada controlado por el usuario en un método `update` o `save`, por que cualquier columna que este en `guarded` podrá actualizarse.

#### Bloquear todos los atributos para asignación masiva

En el ejemplo anterior, los atributos `id` y `password` **no** serán asignados en masa. Todos los otros atributos serán asigables en masa. También puede bloquear **todos** los atributos de la asignación en masa mediante la propiedad `guard`:

	protected $guarded = ['*'];

<a name="insert-update-delete"></a>
## Insertar, actualizar, borrar

Para crear un nuevo registro en la base de datos a partir de un modelo, basta con crear una nueva instancia del modelo y llamar al método `save`.

#### Guardar un nuevo modelo

	$user = new User;

	$user->name = 'John';

	$user->save();

> **Nota:** Por lo general, sus modelos Elocuent tendrán llaves de incremento automático. Sin embargo, si desea especificar sus propias llaves, establezca la propiedad `incrementing` en su modelo a `false`.

También puede utilizar el método `create` para guardar un nuevo modelo en una sola línea. La instancia del modelo insertada le será devuelta del método. Sin embargo, antes de hacerlo, tendrá que especificar cualquer atributo `fillable` o `guarded` en el modelo, ya que todos los modelos Elocuent se protegen contra la asignación en masa.

Después de guardar o crear un nuevo modelo que utiliza id de incremento automático, es posible recuperar el ID accediendo al atributo `id` del objeto:

	$insertedId = $user->id;

#### Configuración del atributo `guarded` en el modelo

	class User extends Model {

		protected $guarded = ['id', 'account_id'];

	}

#### Usando el método "create" en el modelo

	// Crear un nuevo usuario en la base de datos...
	$user = User::create(['name' => 'John']);

	// Recuperar el usuario por los atributos, o crearlo si no existe...
	$user = User::firstOrCreate(['name' => 'John']);

	// Recuperar el usuario por los atributos, o una nueva instancia...
	$user = User::firstOrNew(['name' => 'John']);

#### Actualización de un modelo recuperado

Para actualizar un modelo, es posible recuperarlo, cambiar un atributo, y utilizar el método `save`:

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

#### Guardando un modelo y las relaciones

A veces es posible que desee no sólo salvar un modelo sino también todas sus relaciones. Para ello, puede utilizar el método `push`:

	$user->push();

También puede ejecutar actualizaciones como consultas en un conjunto de modelos:

	$affectedRows = User::where('votes', '>', 100)->update(['status' => 2]);

> **Nota:** Ningún evento del modelo se dispara cuando se actualiza un conjunto de modelos a través del generador de consultas Elocuent.

#### Eliminar un modelo existente

Para borrar un modelo, simplemente llame al método `delete` en la instancia:

	$user = User::find(1);

	$user->delete();

#### Eliminar un modelo existente por llave

	User::destroy(1);

	User::destroy([1, 2, 3]);

	User::destroy(1, 2, 3);

Por supuesto, también puede ejecutar una consulta de eliminación en un conjunto de modelos:

	$affectedRows = User::where('votes', '>', 100)->delete();

#### Actualizar únicamente los timestamps del modelo

Si desea actualizar simplemente las marcas de tiempo en un modelo, puede utilizar el método `touch`:

	$user->touch();

<a name="soft-deleting"></a>
## Borrado blando

Cuando se borra blandamente un modelo, realmente no se quita de la base de datos. En su lugar, la marca de tiempo `deleted_at` es puesta en el registro. Para habilitar el borrado blando, aplíque `SoftDeletes` al modelo.

	use Illuminate\Database\Eloquent\SoftDeletes;

	class User extends Model {

		use SoftDeletes;

		protected $dates = ['deleted_at'];

	}

Para agregar una columna `deleted_at` a su tabla, puede utilizar el método` softDeletes` de la migración:

	$table->softDeletes();

Ahora, cuando llame al método `delete` en el modelo, la columna `deleted_at` se establecerá en la fecha y hora actual. Cuando se consulta un modelo que utiliza borrados suaves, no se incluirán los modelos "borrados" en los resultados de la consulta.

#### Forzar modelos con borrado suave en los resultados

Para forzar los modelos con borrado blando a aparecer en un conjunto de resultados, utilice el método `withTrashed` en la consulta:

	$users = User::withTrashed()->where('account_id', 1)->get();

El método `withTrashed` puede ser utilizado en una relación definida:

	$user->posts()->withTrashed()->get();

Si desea **sólo** recibir los modelos con borrado blando en sus resultados, puede usar el método `onlyTrashed`:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

Para restaurar un modelo con borrado suave a un estado activo, utilice el método `restore`:

	$user->restore();

También puede utilizar el método `restore` en una consulta:

	User::withTrashed()->where('account_id', 1)->restore();

Al igual que con `withTrashed`, el método `restore` también puede usarse en las relaciones:

	$user->posts()->restore();

Si desea eliminar realmente un modelo de la base de datos, puede utilizar el método `forceDelete`:

	$user->forceDelete();

El método `forceDelete` también trabaja en las relaciones:

	$user->posts()->forceDelete();

Para determinar si una instancia del modelo en cuestión se ha borrado blandamente, puede utilizar el método `trashed`:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## Marcas de tiempo

Por omisión, Eloquent mantendrá las columnas `created_at` y `updated_at` en la tabla de su base de datos de forma automática. Sólo añada estas columnas `timestamp` a su tabla y Eloquent se hará cargo del resto. Si usted no desea que Eloquent mantenga estas columnas, agrege la siguiente propiedad a su modelo:

#### Desactivar de timestamps automáticos

	class User extends Model {

		protected $table = 'users';

		public $timestamps = false;

	}

#### Proporcionar un formato personalizado para las marcas de tiempo

Si desea personalizar el formato de sus marcas de tiempo, puede reemplazar el método `getDateFormat` en su modelo:

	class User extends Model {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## Consultas con ámbito

#### Definir una consulta con ámbito

Los ámbitos le permiten reutilizar fácilmente la lógica de consultas en sus modelos. Para definir un ámbito, simplemente anteponga `scope` al método del modelo:

	class User extends Model {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

		public function scopeWomen($query)
		{
			return $query->whereGender('W');
		}

	}

#### Utilizar consultas con ámbito

	$users = User::popular()->women()->orderBy('created_at')->get();

#### Ámbitos dinámicos

A veces es posible que desee definir un ámbito que acepta parámetros. Sólo tiene que añadir sus parámetros a la función de ámbito:

	class User extends Model {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

Luego pase el parámetro a la llamada ámbito:

	$users = User::ofType('member')->get();

<a name="global-scopes"></a>
## Consultas globales con ámbito

A veces es posible que desee definir un ámbito que se aplica a todas las consultas realizadas en un modelo. En esencia, se trata de cómo funciona "borrado blando", una característica própia de Elocuent. Los ámbitos globales se definen usando una combinación de traits de PHP y una implementación de `Illuminate\Database\Eloquent\ScopeInterface`.

En primer lugar, vamos a definir un trait. Para este ejemplo, vamos a utilizar `SoftDeletes` que se incluye con Laravel:

	trait SoftDeletes {

		/**
		 * Iniciar el trait "borrado suave" para un modelo.
		 *
		 * @return void
		 */
		public static function bootSoftDeletes()
		{
			static::addGlobalScope(new SoftDeletingScope);
		}

	}

Si un modelo Elocuent utiliza un trait que tiene un método que coincida con la convención de `bootNameOfTrait`, se llamará el método trait cuando se inicie el modelo Elocuent, dándole la oportunidad de registrar un ámbito global, o hacer cualquier cosa que desee. Un ámbito debe implementar `ScopeInterface`, que especifica dos métodos: `apply` y `remove`.

El método `apply` recibe un objeto generador de consultas `Illuminate\Database\Elocuent\Builder` y el `Modelo` al que se aplica, y es responsable adicionar cualquier clausula `where` adicional que el ámbito desee agregar. El método `remove` también recibe un objeto `Builder` y un `Modelo` y es responsable de reversar la acción tomada por `apply`. En otras palabras, `remove` debe quitar las cláusulas `where` añadidas (o cualquier otra cláusula). Así, para nuestro `SoftDeletingScope`, los métodos buscan algo como esto:

	/**
	 * Aplica el ámbito al "query builder" dado.
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function apply(Builder $builder, Model $model)
	{
		$builder->whereNull($model->getQualifiedDeletedAtColumn());

		$this->extend($builder);
	}

	/**
	 * Elimina el ámbito del "query builder" dado.
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function remove(Builder $builder, Model $model)
	{
		$column = $model->getQualifiedDeletedAtColumn();

		$query = $builder->getQuery();

		foreach ((array) $query->wheres as $key => $where)
		{
		    // Si la cláusula where es una restricción de eliminación de fecha blanda, 
		    // lo eliminaremos de la consulta y se restableceran las llaves en los wheres. Esto 
		    // permite al desarrollador incluir modelos borrados en un conjunto de resultados de la 
		    // relación que es cargada posteriormente.
			if ($this->isSoftDeleteConstraint($where, $column))
			{
				unset($query->wheres[$key]);

				$query->wheres = array_values($query->wheres);
			}
		}
	}

<a name="relationships"></a>
## Las relaciones

Por supuesto, las tablas en la base de datos probablemente están relacionados entre sí. Por ejemplo, una entrada de blog puede tener muchos comentarios, o una orden podría estar relacionada con el usuario que la colocó. Eloquent hace que la gestión y el trabajo con estas relaciones sea fácil. Laravel soporta muchos tipos de relaciones:

- [Uno a uno](#one-to-one)
- [Uno a muchos](#one-to-many)
- [Muchos a muchos](#many-to-many)
- [Tiene muchos a través de](#has-many-through)
- [Relaciones polimórficas](#polymorphic-relations)
- [Relaciones polimórficas muchos a muchos](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### Uno a uno

#### Definir una relación uno a uno

Una relación uno-a-uno es una relación muy básica. Por ejemplo, un modelo `User` podría tener un `Phone`. Podemos definir esta relación en Elocuent:

	class User extends Model {

		public function phone()
		{
			return $this->hasOne('App\Phone');
		}

	}

El primer argumento pasado al método `hasOne` es el nombre del modelo relacionado. Una vez definida la relación, podemos recuperarla usando las [propiedades dinámicas](#dynamic-properties) de Elocuent:

	$phone = User::find(1)->phone;

El SQL realizado por esta declaración será el siguiente:

	select * from users where id = 1

	select * from phones where user_id = 1

Tenga en cuenta que Elocuent asume la llave externa de la relación basada en el nombre del modelo. En este caso, el modelo `Phone` asume que debe usar la llave externa `user_id`. Si desea anular esta convención, es posible pasar un segundo argumento del método `hasOne`. Además, es posible pasar un tercer argumento al método para especificar qué columna local se debe utilizar para la asociación:

	return $this->hasOne('App\Phone', 'llave_externa');

	return $this->hasOne('App\Phone', 'llave_externa', 'llave_local');

#### Definir del Inverso de una Relación

Para definir el inverso de la relación en el modelo `Phone`, utilizamos el método `belongsTo`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

En el ejemplo anterior, Elocuent buscará una columna `user_id` sobre la tabla `phones`. Si desea definir una columna diferente para la llave externa, puede pasarla como segundo argumento al método `belongsTo`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'llave_local');
		}

	}

Además, puede pasar un tercer parámetro que especifica el nombre de la columna asociada en la tabla padre:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'llave_local', 'llave_padre');
		}

	}

<a name="one-to-many"></a>
### Uno a muchos

Un ejemplo de una relación uno-a-muchos es un blog que "tiene muchos" comentarios. Podemos modelar esta relación, así:

	class Post extends Model {

		public function comments()
		{
			return $this->hasMany('App\Comment');
		}

	}

Ahora podemos acceder a los comentarios del blog a través de las [propiedades dinámicas](#dynamic-properties):

	$comments = Post::find(1)->comments;

Si necesita añadir más restricciones a los comentarios que se recuperan, puede llamar al método `comments` y seguir encadenando condiciones:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

Una vez más, es posible anular la llave externa convencional pasando un segundo argumento del método `hasMany`. Y, como en la relación `hasOne`, la columna local puede también ser especificada:

	return $this->hasMany('App\Comment', 'llave_externa');

	return $this->hasMany('App\Comment', 'llave_externa', 'llave_local');

#### Definir inverso de la relación

Para definir el inverso de la relación en el modelo `Comment`, utilizamos el método `belongsTo`:

	class Comment extends Model {

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

<a name="many-to-many"></a>
### Muchos a muchos

Las relaciones muchos-a-muchos son un tipo más complicado de relación. Un ejemplo de esta relación es un usuario con muchos roles, donde los roles también son compartidos por otros usuarios. Por ejemplo, muchos usuarios pueden tener el rol de "Admin". Se necesitan tres tablas para esta relación: `users`, `roles`, y` role_user`. La tabla `role_user` se deriva del orden alfabético de los nombres de los modelos relacionados, y debe tener las columnas `user_id` y `role_id`.

Podemos definir una relación muchos-a-muchos con el método `belongsToMany`:

	class User extends Model {

		public function roles()
		{
			return $this->belongsToMany('App\Role');
		}

	}

Ahora, podemos recuperar los roles a través del modelo `User`:

	$roles = User::find(1)->roles;

Si desea utilizar un nombre de tabla no convencional para su tabla pivote, puede pasarlo como segundo argumento al método `belongsToMany`:

	return $this->belongsToMany('App\Role', 'user_roles');

También puede cambiar las llaves asociadas convencionales:

	return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'foo_id');

Por supuesto, también puede definir el inverso de la relación en el modelo `Role`:

	class Role extends Model {

		public function users()
		{
			return $this->belongsToMany('App\User');
		}

	}

<a name="has-many-through"></a>
### Tiene muchos a través de

La relación "tiene muchos a través de" proporciona un atajo cómodo para visitar parientes lejanos a través de una relación intermedia. Por ejemplo, un modelo `Country` podría tener muchos `Post` a través de un modelo `User`. Las tablas de esta relación podría tener este aspecto:

	countries
		id - integer
		name - string

	users
		id - integer
		country_id - integer
		name - string

	posts
		id - integer
		user_id - integer
		title - string

A pesar de que la tabla `posts` no contiene una columna `country_id`, la relación `hasManyThrough` nos permitirá acceder a los posts de un país a través de `$country->posts`. Vamos a definir la relación:

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User');
		}

	}

Si desea especificar manualmente las llaves de la relación, puede pasarlas como el tercer y cuarto argumentos del método:

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
		}

	}

<a name="polymorphic-relations"></a>
### Relaciones polimórficas

Las relaciones polimórficas permiten a un modelo pertenecer a más de un modelo, en una sola asociación. Por ejemplo, usted podría tener un modelo de foto que pertenece tanto a un modelo de personal o a un modelo de orden. Queremos definir esta relación, así:

	class Photo extends Model {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Model {

		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}

	}

	class Order extends Model {

		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}

	}

#### Obtener una relación polimórfica

Ahora, podemos recuperar las fotos, ya sea para un miembro del personal o una orden:

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

#### Obtener el propietario de una relación polimórfica

Sin embargo, la verdadera magia "polimórfica" es cuando se accede al personal u orden del modelo `Photo`:

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

El relación `imageable` en el modelo `Photo` devolverá una instancia de `Staff` o de `Order`, dependiendo de qué tipo de modelo es propietario de la foto.

#### Estructura de la tabla en una relación polimórfica

Para ayudar a entender cómo funciona esto, vamos a explorar la estructura de base de datos para una relación polimórfica:

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

Los campos clave para notar aquí son `imageable_id` y `imageable_type` sobre la tabla `photos`. El ID contendrá el valor del ID de, en este ejemplo, el personal o la orden propietarios, mientras que el tipo contendrá el nombre de la clase del modelo propietario. Esto es lo que permite que el ORM pueda determinar qué tipo debe ser el dueño del modelo para volver al acceder a la relación `imageable`.

<a name="many-to-many-polymorphic-relations"></a>
### Relación polimórfica muchos a muchos

#### Estructura de la tabla para relaciones polimórficas muchos a muchos 

Además de las relaciones polimórficas tradicionales, también puede especificar relaciones polimórficas muchos-a-muchos. Por ejemplo, un modelo blog `Post` y un modelo `Video` podrían compartir una relación polimórfica a un modelo `tag`. En primer lugar, vamos a examinar la estructura de la tabla:

	posts
		id - integer
		name - string

	videos
		id - integer
		name - string

	tags
		id - integer
		name - string

	taggables
		tag_id - integer
		taggable_id - integer
		taggable_type - string

A continuación, estamos listos para configurar las relaciones en el modelo. En los modelos `Video` y `Post` ambos tienen una relación `morphToMany` a través de un método `tags`:

	class Post extends Model {

		public function tags()
		{
			return $this->morphToMany('App\Tag', 'taggable');
		}

	}

El modelo `Tag` puede definir un método para cada uno de sus relaciones:

	class Tag extends Model {

		public function posts()
		{
			return $this->morphedByMany('App\Post', 'taggable');
		}

		public function videos()
		{
			return $this->morphedByMany('App\Video', 'taggable');
		}

	}

<a name="querying-relations"></a>
## Consulta de relaciones

#### Consulta de relaciones a la hora de seleccionar

Al acceder a los registros de un modelo, es posible que desee limitar sus resultados sobre la base de la existencia de una relación. Por ejemplo, usted desea recuperar todos los posts de blogs que tienen al menos un comentario. Para ello, puede utilizar el método `has`:

	$posts = Post::has('comments')->get();

También puede especificar un operador y un conteo:

	$posts = Post::has('comments', '>=', 3)->get();

Pueden construirse declaraciones `has` anidadas usando la notación "dot":

	$posts = Post::has('comments.votes')->get();

Si necesita más potencia, puede utilizar los métodos `whereHas` y `orWhereHas` para ponerle al "where" condiciones `has` en sus consultas:

	$posts = Post::whereHas('comments', function($q)
	{
		$q->where('content', 'like', 'foo%');

	})->get();

<a name="dynamic-properties"></a>
### Propiedades dinámicas

Eloquent le permite acceder a sus relaciones a través de las propiedades dinámicas. Eloquent cargará automáticamente la relación por usted, y es aún lo suficientemente inteligente como para saber si llamar al `get` (para relaciones uno-a-muchos) o el método `first` (para relaciones uno-a-uno). A continuación, será accesible a través de una propiedad dinámica con el mismo nombre de la relación. Por ejemplo, con el siguiente modelo `$phone`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

	$phone = Phone::find(1);

En lugar mostrar el correo electrónico del usuario así:

	echo $phone->user()->first()->email;

Puede ser simplificado a simplemente::

	echo $phone->user->email;

> **Nota:** Las relaciones que devuelven varios resultados devolverán una instancia de la classe `Illuminate\Database\Eloquent\Collection`.

<a name="eager-loading"></a>
## Carga impaciente

Existe la carga impaciente para aliviar el problema de la consulta N + 1. Por ejemplo, considere un modelo `Book` que se relaciona con `author`. La relación se define de este modo:

	class Book extends Model {

		public function author()
		{
			return $this->belongsTo('App\Author');
		}

	}

Ahora, considere el siguiente código:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

En este bucle se ejecutará 1 consulta para recuperar todos los libros de la tabla, y luego otra consulta para cada libro para recuperar el autor. Por lo tanto, si tenemos 25 libros, este bucle se ejecutaría 26 consultas.

Afortunadamente, podemos utilizar la carga impaciente para reducir drásticamente el número de consultas. Las relaciones que se deben cargar impacientemente se pueden especificar mediante el método `with`:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

En el bucle anterior, se ejecutarán sólo dos consultas:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

El uso racional de la carga impaciente puede aumentar drásticamente el rendimiento de la aplicación.

Por supuesto, usted puede cargar impacientemente múltiples relaciones a la vez:

	$books = Book::with('author', 'publisher')->get();

Es posible que cargar impacientemente relaciones anidadas:

	$books = Book::with('author.contacts')->get();

En el ejemplo anterior, la relación `author` será cargada impacientemente, y también será cargada la relación `contacts` del autor.

### Restringir la carga impaciente

A veces es posible que desee cargar impacientemente una relación, pero también puede especificar una condición para la carga impaciente. He aquí un ejemplo:

	$users = User::with(['posts' => function($query)
	{
		$query->where('title', 'like', '%first%');

	}])->get();

En este ejemplo, estamos cargando impacientement los mensajes del usuario, pero sólo si la columna de título del post contiene la palabra "first".

Por supuesto, los closures de carga impaciente no se limitan a "restricciones". Usted también puede ordenarlas:

	$users = User::with(['posts' => function($query)
	{
		$query->orderBy('created_at', 'desc');

	}])->get();

### Carga impaciente tardía

También es posible cargar impacientemente modelos relacionados directamente de un modelo en una colección ya existente. Esto puede ser útil al momento de decidir si se debe cargan dinámicamente modelos relacionados o no, o en combinación con el almacenamiento en el caché.

	$books = Book::all();

	$books->load('author', 'publisher');

También puede pasar un closure para establecer restricciones a la consulta:

	$books->load(['author' => function($query)
	{
		$query->orderBy('published_date', 'asc');
	}]);

<a name="inserting-related-models"></a>
## Insertar modelos relacionados

#### Vincular un modelo relacionado

A menudo se necesita insertar nuevos modelos relacionados. Por ejemplo, puede que desee insertar un nuevo comentario a un post. En lugar de establecer manualmente la llave externa `post_id` en el modelo, es posible insertar directamente el comentario desde el modelo padre `Post`:

	$comment = new Comment(['message' => 'A new comment.']);

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

En este ejemplo, el campo `post_id` se ajustará automáticamente en el comentario insertado.

Si necesita guardar varios modelos relacionados:

	$comments = [
		new Comment(['message' => 'A new comment.']),
		new Comment(['message' => 'Another comment.']),
		new Comment(['message' => 'The latest comment.'])
	];

	$post = Post::find(1);

	$post->comments()->saveMany($comments);

### Asociar modelos (Belongs To)

Al actualizar una relación `belongsTo`, puede utilizar el método `associate`. Este método establecerá la llave externa en el modelo hijo:

	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### Insertar modelos asociados (Many To Many)

También puede insertar modelos relacionados cuando se trabaja con ralciones muchos-a-muchos. Vamos a continuar utilizando como ejemplos nuestros modelos `User` y `Role`. Podemos conectar fácilmente nuevos roles a un usuario utilizando método `attach`:

#### Vincular modelos (Many To Many)

	$user = User::find(1);

	$user->roles()->attach(1);

También puede pasar un arreglo de atributos que se deben almacenar en la tabla pivote de la relación:

	$user->roles()->attach(1, ['expires' => $expires]);

Por supuesto, lo contrario de `attach` es `detach`:

	$user->roles()->detach(1);

Tanto `attach` como `detach` toman arreglos de IDs como entrada:

	$user = User::find(1);

	$user->roles()->detach([1, 2, 3]);

	$user->roles()->attach([1 => ['attribute1' => 'value1'], 2, 3]);

#### Sincronizar para adjuntar modelos (Many To Many)

También puede utilizar el método `sync` para adjuntar modelos relacionados. El método `sync` acepta un arreglo de IDs para poner en la tabla pivote. Después de esta operación, sólo los IDs del arreglo estarán en la tabla pivote para el modelo:

	$user->roles()->sync([1, 2, 3]);

#### Añadir datos pivote cuando se sincroniza

También puede asociar otros valores a la tabla pivote con los IDs dados:

	$user->roles()->sync([1 => ['expires' => true]]);

A veces es posible que desee crear un nuevo modelo relacionado y vincularlo en un solo comando. Para esta operación, es posible utilizar el método `save`:

	$role = new Role(['name' => 'Editor']);

	User::find(1)->roles()->save($role);

En este ejemplo, el nuevo modelo `Role` se guardará y se vincula al modelo de usuario. También puede pasar un arreglo de atributos para poner en la tabla pivote para esta operación:

	User::find(1)->roles()->save($role, ['expires' => $expires]);

<a name="touching-parent-timestamps"></a>
## Actualizar marcas de tiempo en el padre

Cuando un modelo pertence a otro modelo, como un `Comment` que pertenece a un `Post`, a menudo es útil actualizar la marca de tiempo del padre cuando se actualiza el modelo hijo. Por ejemplo, cuando se actualiza un modelo `Comment`, es posible que desee actualizar automáticamente el "timestamp" `updated_at` del padre `Post`. Eloquent lo hace fácil. Sólo tiene que añadir la propiedad `touches` que contiene los nombres de las relaciones con el modelo hijo:

	class Comment extends Model {

		protected $touches = ['post'];

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

Ahora, cuando se actualiza un `Comment`, el `Post` padre tendrá actualizada su columna `updated_at`:

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## Trabajar tablas pivote

Como ya ha aprendido, trabajar con relaciones muchos-a-muchos requiere la presencia de una tabla intermedia. Elocuent ofrece algunas maneras muy útiles de interactuar con esta tabla. Por ejemplo, supongamos que nuestro objeto `User` tiene muchos objetos `Role` relacionados. Después de acceder a esta relación, podemos acceder a la tabla `pivot` en los modelos:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Observe que a cada modelo `Role` recuperado se le asigna automáticamente un atributo `pivot`. Este atributo contiene un modelo que representa la tabla intermedia, y puede ser utilizado como cualquier otro modelo Elocuent.

Por omisión, sólo las llaves estarán presentes en el objeto `pivot`. Si su tabla pivote contiene atributos adicionales, debe especificarlos en la definición de la relación:

	return $this->belongsToMany('App\Role')->withPivot('foo', 'bar');

Ahora, los atributos `foo` y `bar` serán accesibles en nuestro objeto `pivot` para el modelo `Role`.

Si desea que la tabla pivote mantenga automáticamente las marcas de tiempo `created_at` y `updated_at`, utilice el método `withTimestamps` en la definición de la relación:

	return $this->belongsToMany('App\Role')->withTimestamps();

#### Borrar registros en una tabla pivote

Para borrar todos los registros de la tabla pivote para un modelo, puede utilizar el método `detach`:

	User::find(1)->roles()->detach();

Tenga en cuenta que esta operación no elimina los registros de la tabla `roles`, sólo de la tabla pivote.

#### Actualizar un registro en una tabla pivote

A veces puede que tenga que actualizar su tabla pivote sin desvincularla. Si desea actualizar su tabla pivote ahí mismo puede usar el método `updateExistingPivot` de este modo:

	User::find(1)->roles()->updateExistingPivot($roleId, $attributes);

#### Definir de uno modelo pivote personalizado

Laravel también le permite definir un modelo pivote personalizado. Para definir un modelo personalizado, primero debe crear su propia clase de modelo "Base" que se extiende `Eloquent`. En sus otros modelos Eloquent, extienda este modelo personalizado en lugar de la base predeterminada `Eloquent`. En el modelo base, agregue la siguiente función que devuelve una instancia de su modelo pivote personalizado:

	public function newPivot(Model $parent, array $attributes, $table, $exists)
	{
		return new YourCustomPivot($parent, $attributes, $table, $exists);
	}

<a name="collections"></a>
## Colecciones

Todos los conjuntos de resultados múltiples devueltos por Eloquent, ya sea a través del método `get` o una `relationship`, devolverán un objeto colección. Este objeto implementa la interfaz `IteratorAggregate` de PHP para que pueda ser iterada como una arreglo. Sin embargo, este objeto también tiene una variedad de métodos útiles para trabajar con conjuntos de resultados.

#### Comprobar si una colección tiene una llave

Por ejemplo, podemos determinar si un conjunto de resultados contiene una llave primaria dada usando el método `contains`:

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Las colecciones también se pueden convertir en un arreglo o JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

Si una colección se forza a cadena, se devuelve como JSON:

	$roles = (string) User::find(1)->roles;

#### Iterar colecciones

Las colecciones Eloquent también contienen algunos métodos útiles para bucles y para filtrar los elementos que contengan:

	$roles = $user->roles->each(function($role)
	{
		//
	});

#### Filtrar colecciones

Al filtrar colecciones, la llamada de retorno proporcionada será utilizada como llamada de retorno para [array_filter] (http://php.net/manual/en/function.array-filter.php).

	$users = $users->filter(function($user)
	{
		return $user->isAdmin();
	});

> **Nota:** Al filtrar una colección y convertirla a JSON, intente llamar primero a la función `values` para restablecer las llaves del arreglo.

#### Aplicar una llamada de retorno a cada objeto de la colección

	$roles = User::find(1)->roles;

	$roles->each(function($role)
	{
		//
	});

#### Ordernar una collección por valor

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});

#### Ordenar una collecion por valor

	$roles = $roles->sortBy('created_at');

#### Retirnar un tipo personalizado de colección

A veces, es posible que desee devolver un objeto Colección personalizado con sus propios métodos añadidos. Puede especificar esto en su modelo Elocuent reemplazando el método `newCollection`:

	class User extends Model {

		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Accesores y mutadores

#### Definir un accesor

Eloquent proporciona una manera conveniente para transformar los atributos del modelo al accederlos o cambiarlos. Basta con definir un método `getFooAttribute` en el modelo para declarar un accesor. Tenga en cuenta que los métodos deben seguir la convención CamelCase, a pesar de que las columnas en la bases de datos sean snake-case:

	class User extends Model {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

En el ejemplo anterior, la columna `first_name` tiene un accesor. Tenga en cuenta que el valor del atributo se pasa al accesor.

#### Definir un mutador 

Los mutators se declaran de forma similar:

	class User extends Model {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Mutadores de fecha

Por omisión, Eloquent convertirá las columnas `created_at` y `updated_at` a instancias de [Carbon](https://github.com/briannesbitt/Carbon), que ofrece una variedad de métodos útiles, y extiende la clase la nativa de PHP `DateTime`.

Puede personalizar qué campos se mutan de forma automática, y desactivar incluso completamente esta mutación, reemplazando el método `getDates` del modelo:

	public function getDates()
	{
		return ['created_at'];
	}

Cuando una columna se considera una fecha, es posible fijar su valor a una marca de tiempo UNIX, cadena de fecha (`Y-m-d`), cadena de fecha y hora, y por supuesto a una instancia `DateTime`/ `Carbon`.

Para deshabilitar totalmente las mutaciones de fecha, devuelva una arreglo vacío del método `getDates`:

	public function getDates()
	{
		return [];
	}

<a name="attribute-casting"></a>
## Forzar tipos

Si usted tiene algunos atributos que desea convertir siempre a otro tipo de datos, usted puede añadir el atributo a la propiedad `casts` de su modelo. De lo contrario, tendrá que definir un mutador para cada uno de los atributos, esto puede consumir bastante tiempo. He aquí un ejemplo del uso de la propiedad `casts`:

	/**
	 * Los atributos que deben ser forzados a tipos nativos.
	 *
	 * @var array
	 */
	protected $casts = [
		'is_admin' => 'boolean',
	];

Ahora el atributo `is_admin` siempre será convertido a un valor lógico cuando acceda a el, incluso si el valor subyacente se almacena en la base de datos como un entero. Otros tipos de soportados son: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object` y `array`.

El forzado a `array` es especialmente útil para trabajar con columnas que se almacenan como JSON serializado. Por ejemplo, si su base de datos tiene un campo de tipo TEXT que contiene JSON serializado, añadiendo el forzado a `array` ese atributo deserializara automáticamente el atributo a un arreglo de PHP cuando se accede a él en su modelo Eloquent:

	/**
	 * Los atributos deben ser forzados a tipos nativos.
	 *
	 * @var array
	 */
	protected $casts = [
		'options' => 'array',
	];

Ahora, cuando usted utiliza el modelo Eloquent:

	$user = User::find(1);

	// $options is an array...
	$options = $user->options;

	// options is automatically serialized back to JSON...
	$user->options = ['foo' => 'bar'];

<a name="model-events"></a>
## Eventos de modelos

Los modelos de Eloquent disparan varios eventos, lo que le permite conectarse en varios puntos en el ciclo de vida del modelo utilizando los siguientes métodos: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Cada vez que un nuevo modelo se guarda por primera vez, se dispararán los eventos `creating` y `created`. Si un modelo no es nuevo el método `save` se llama, los métodos `updating`/`updated` se disparan. En ambos casos, los métodos `saving`/`saved` se dispararán.

#### Cancelar la operaciones de guardado vía eventos

Si se devuelve `false` desde los eventos `creating`, `updating`, `saving`, o `deleting`, se cancelará la acción:

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

#### Donde registrar los oidores de eventos

Su `EventServiceProvider` sirve como un lugar conveniente para registrar sus enlaces de eventos de modelo. Por ejemplo:

	/**
	 * Registrar cualquier otro evento para su aplicación.
	 *
	 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
	 * @return void
	 */
	public function boot(DispatcherContract $events)
	{
		parent::boot($events);

		User::creating(function($user)
		{
			//
		});
	}

<a name="model-observers"></a>
## Observadores de modelos

Para consolidar el manejo de eventos del modelo, es posible registrar un observador de modelo. Una clase observador puede tener métodos que corresponden a los diversos eventos del modelo. Por ejemplo, los métodos `creating`, `updating`, `saving` pueden estar en un observador, además de cualquier otro nombre del evento del modelo.

Así, por ejemplo, un observador de modelo podría tener este aspecto:

	class UserObserver {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

Usted puede registrar una instancia del observador usando el método `observe`:

	User::observe(new UserObserver);

<a name="model-url-generation"></a>
## Generar URL para modelos

Cuando pasa un modelo a los métodos `route` o `action`, es la llave principal la que se inserta en la URI generada. Por ejemplo:

	Route::get('user/{user}', 'UserController@show');

	action('UserController@show', [$user]);

En este ejemplo, la propiedad `$user->id` se inserta en el lugar de `{user}` de la URL generada. Sin embargo, si desea utilizar otra propiedad en lugar del ID, puede reemplazar el método `getRouteKey` de su modelo:

	public function getRouteKey()
	{
		return $this->slug;
	}

<a name="converting-to-arrays-or-json"></a>
## Convertir a JSON / Arreglos

#### Convertir modelos en arreglos

Cuando construye API de JSON, es posible que a menudo necesite convertir sus modelos y relaciones a arreglos o JSON. Así, Eloquent incluye métodos para hacerlo. Para convertir un modelo y su relación cargada a un arreglo, puede utilizar el método `toArray`:

	$user = User::with('roles')->first();

	return $user->toArray();

Tenga en cuenta que las colecciones de modelos enteras también pueden ser convertidas a arrays:

	return User::all()->toArray();

#### Convertir un modelo a JSON

Para convertir un modelo a JSON, puede utilizar el método `toJson`:

	return User::find(1)->toJson();

#### Devolviendo un modelo desde una ruta

Tenga en cuenta que cuando un modelo o colección es forzado a cadena, que se convertirá en JSON, lo que significa que puede devolver objetos Eloquent directamente de las rutas de su aplicación!

	Route::get('users', function()
	{
		return User::all();
	});

#### Ocultando atributos para las conversión a arreglos o JSON

A veces es posible que desee limitar los atributos que se incluyen en la forma arreglo o JSON de su modelo, como contraseñas. Para ello, agregue una definición de la propiedad `hidden` a su modelo:

	class User extends Model {

		protected $hidden = ['password'];

	}

> **Nota:** Cuando oculte relaciones, use el nombre del **método** de la relación, no el nombre del acceso dinámico de la relación.

Alternativamente, puede usar la propiedad `visible` para definir una lista blanca:

	protected $visible = ['first_name', 'last_name'];

<a name="array-appends"></a>
De vez en cuando, es posible que deba agregar un arreglo de atributos que no tienen una columna correspondiente en su base de datos. Para ello, basta con definir un accesor para el valor:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

Una vez que haya creado el accesor, sólo tiene que añadir el valor a la propiedad `appends` del modelo:

	protected $appends = ['is_admin'];

Una vez que el atributo ha sido añadido a la lista de `appends`, será incluido en las formas del modelo de arreglo y JSON. Los atributos en el arreglo `appends` respetan las configuraciones `visible` y `hidden` del modelo.
