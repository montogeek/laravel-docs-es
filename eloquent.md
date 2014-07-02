# Eloquent ORM

- [Introducción](#introduction)
- [Uso básico](#basic-usage)
- [Asignación en masa](#mass-assignment)
- [Insertar, Actualizar, Eliminar](#insert-update-delete)
- [Eliminación flexible](#soft-deleting)
- [Marcas de tiempo](#timestamps)
- [Consultas con ámbito](#query-scopes)
- [Relaciones](#relationships)
- [Consultas sobre relaciones](#querying-relations)
- [Carga impaciente](#eager-loading)
- [Insertando modelos relacionados](#inserting-related-models)
- [Actualizando marcas de tiempo de relaciones padre](#touching-parent-timestamps)
- [Trabajando con tablas intermedias](#working-with-pivot-tables)
- [Colecciones](#collections)
- [Descriptores de acceso y Mutadores](#accessors-and-mutators)
- [Mutadores de fecha](#date-mutators)
- [Eventos de modelos](#model-events)
- [Observadores de modelos](#model-observers)
- [Convirtiendo a Arreglos / JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Introducción

El ORM Eloquent incluído con Laravel provee una implementación de ActiveRecord hermosa y sencilla para trabajar con tu base de datos. Cada tabla de la base de datos tiene un "Modelo" correspondiente que es usado para interactuar con la tabla.

Antes de empezar, asegúrate de configurar la conexión a la base de datos en `app/config/database.php`.

<a name="basic-usage"></a>
## Uso básico

Para empezar, crea un modelo Eloquent. Los modelos comúmente se guardan en el directorio `app/models`, pero eres libre de guardarlos en cualquier directorio que se puede autocargar a través de tu archivo `composer.json`.

#### Definiendo un modelo Eloquent

	class User extends Eloquent {}

Observa que no le especificamos a Eloquent cual tabla se usará para nuestro modelo `User`. El nombre en plural y minúsculas será usado como nombre de la tabla a menos que se espeficique otra explicítamente. Así, en este aso, Eloquent asumirá que el modelo	 `User` guardará los datos en la tabla `users`. También puedes especificar una tabla personalizada definiendo la propiedad `table` en tu modelo:

	class User extends Eloquent {

		protected $table = 'my_users';

	}

> **Nota:** Eloquent también asumirá que cada tabla tiene una llame primaria llamada `id`. Puedes definir la propiedad `primaryKey` para sobreescribir esta convención. Igualmente, puedes definir la propiedad  `connection` para sobreescribir el nombre de la conexión de base de datos que utilizará el modelo.

Una vez el modelo esté definido, estás listo para empezar a obtener y crear registros en tu tabla. Es necesario crear las columnas `updated_at` y `created_at` en tu tabla, sino quieres tener estás columnas mantenidas automáticamente, establece la propiedad `$timestamps` en tu modelo a `false`.

#### Obteniendo todos los registros

	$users = User::all();

#### Obteniendo un modelo por su llave primaria

	$user = User::find(1);

	var_dump($user->name);

> **Nota:** Todos los métodos disponibles en el [constructor de consultas](/page/queries) también están disponibles cuando se consulta con modelos de Eloquent.

#### Obteniendo un modelo por su llave primaria o lanzando una excepción

Algunas veces puedes desear lanzar una excepción si un modelo no es encontrado, permitiendote atrapar la excepción usando el manejador `App::error` y mostrando una página de error 404.

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

Para regisrar un manejador de errores, escucha la excepción `ModelNotFoundException`

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	App::error(function(ModelNotFoundException $e)
	{
		return Response::make('Not Found', 404);
	});

#### Consultando usando los modelos de Eloquent

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

Por supuesto, también puedes usar las funciones de agregados del constructor de consultas.

#### Agregados de Eloquent

	$count = User::where('votes', '>', 100)->count();

Si no puedes construir una consulta a través de la interfaz fluída, siénte libre de usar el método `whereRaw`:

	$users = User::whereRaw('age > ? and votes = 100', array(25))->get();

#### Dividiendo resultados

Si necesitar procesar muchos (miles) de registros de Eloquent, usar el método `chunk` te permitirá hacerlo sin consumir toda tu memoria RAM:

	User::chunk(200, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});

El primer argumento pasado al método es el número de registros que deseas recibir por "porción". La Clausura pasado como segundo parámetro será ejecutada para cada porción que es traída de la base de datos.

#### Especificando la conexión de la consulta

Tambien puedes especificar que conexión de base de datos debería ser usada cuando se ejecuta una consulta de Eloquent. Simplemente usa el método `on`:

	$user = User::on('connection-name')->find(1);

<a name="mass-assignment"></a>
## Asignación en masa

Cuando creas un nuevo registro, pasas un arreglo de atributos al constructor del modelo. Esos atributos luego son asignados al modelo a través de una asignación en masa. Esto es conveniente; sin embargo, puede ser un **grave y serio** problema de seguridad pasar ciegamente la entrada de datos del usuario al modelo. Si la entrada de datos del usuario es pasada ciegamente al modelo, el usuario está en la libertad de modificar **cualquiera** y **todos** los atributos del modelo. Por esta razón, todos los modelos de Eloquent están protegidos de la asignación en masa de forma predeterminada.

Para empezar, establece las propiedades `fillable` o `guarded` en tu modelo.

La propiedad `fillable` especifica que atributos deberían poder asignarse en masa. Esto puede ser establecido a nivel de la clase o en una instancia.

#### Definiendo los atributos que se puedan asignar en un modelo

	class User extends Eloquent {

		protected $fillable = array('first_name', 'last_name', 'email');

	}

En este ejemplo, solo los tres atributos listados podrán ser asignados en masa.

El inverso de `fillable` es `guarded`, y sirve como una "lista negra" en vez de una "lista blanca."

#### Definiendo los atributos asegurados en un modelo

	class User extends Eloquent {

		protected $guarded = array('id', 'password');

	}

En el anterior ejemplo, los atributos `id` y `password` **no** pueden ser asignados en masa. El resto de atributos se podrán asignar en masa. También puedes bloquear **todos** los atributos de la asignación en masa usando la propiedad `guarded`:

#### Bloqueando todos los atributos de la asignación en masa

	protected $guarded = array('*');

<a name="insert-update-delete"></a>
## Insertar, Actualizar, Eliminar

Para crear un nuevo registro en la base dedatos de un modelo, simplemente crea una nueva instancia del modelo y llama el método `save`:

#### Guardando un nuevo modelo

	$user = new User;

	$user->name = 'John';

	$user->save();

> **Nota:** Normalmente, tus modelos de Eloquent tendrán llaves autoincrementables. Sin embargo, si deseas especificar tus propias llaves, establece la propiedad `incrementing` en `false`.

También puedes usar el método `create` para guardar un nnuevo modelo en una sola línea. La instancia del modelo insertado será retornado del método. Sin embargo, antes de hacerlo, necesitas especificar el atributo `fillabe` o `guarded` en tu modelo, como todos los modelos Eloquent protegido contra asignación en masa.

Luego de guardar o creat el nuevo modelo que usa IDs autoinccrementable, puedes obtener el ID accediendo al atributo `id` del modelo:

	$insertedId = $user->id;

#### Configurando los atributos bloqueados de la asignación en masa en el modelo

	class User extends Eloquent {

		protected $guarded = array('id', 'account_id');

	}

#### Usando el método create del modelo

	// Create a new user in the database...
	$user = User::create(array('name' => 'John'));

	// Retrieve the user by the attributes, or create it if it doesn't exist...
	$user = User::firstOrCreate(array('name' => 'John'));

	// Retrieve the user by the attributes, or instantiate a new instance...
	$user = User::firstOrNew(array('name' => 'John'));

para actualiar un modelo, puedes obtenerlo, cambiar un atributo, y usar el método `save`:

#### Actualizando un modelo obtenido

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

Algunas veces deseas no solo guardar el modelo, sino también todas sus relaciones. Para hacerlo, puedes usar el método `push`:

#### Guardando un modelo y sus relaciones

	$user->push();

También puedes ejecutar actualizaciones como consultas contra un conjunto de modelos:

	$affectedRows = User::where('votes', '>', 100)->update(array('status' => 2));

> **Nota:** No se disparan eventos cuando se actualizan un conjunto de modelos a través del constructor de consultas de Eloquent.

#### Eliminando un modelo existente

Para eliminar un modelo, simplemente llama el método `delete` en la instancia:

	$user = User::find(1);

	$user->delete();

#### Deliminando un modelo existente por su llave

	User::destroy(1);

	User::destroy(array(1, 2, 3));

	User::destroy(1, 2, 3);

Por supuesto, también puedes ejecutar una consulta para eliminar un conjunto de modelos:

	$affectedRows = User::where('votes', '>', 100)->delete();

#### Actualizando solo las marcas de tiempo del modelo

Si deseas simplemente actualizar lar marcas de tiempo en un modelo, puedes usar el método `touch`:

	$user->touch();

<a name="soft-deleting"></a>
## Eliminación flexible

Cuando elimines de forma flexible un modelo, realmente no se está eliminando de tu base de datos. En realidad, una marca de tiempo `deleted_at` is establecida en el registro. Para habilitar eliminación flexible en un modelo, especifica la propiedad `softDelete` en el modelo:

	class User extends Eloquent {

		protected $softDelete = true;

	}

Para agregar la columna `deleted_at` a tu tabla, puedes usar el método `softDeletes` en la migración:

	$table->softDeletes();

Ahora, cuando llames el método `delete` en un modelo, en la columna `deleted_at` será establecida la marca de tiempo actual. Cuando consultes un modelo que use eliminación flexible, los  registros "eliminados" no serán incluídos en los resultados de la consulta. Para obligar que los registros eliminados aparezcan en los resultados, usa el método `withTrashed` en la consulta:

#### Obligando registros eliminados flexiblemente en resultados

	$users = User::withTrashed()->where('account_id', 1)->get();

El método `withTrashed` puedes ser usado en una relación definida:

	$user->posts()->withTrashed()->get();

Si **solamente** deseas obtener los registros eliminados flexiblemente en tus resultados, puedes usar el método `onlyTrashed`:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

Para restaurar registros eliminados flexiblemente al estado activo, usa el método `restore`:

	$user->restore();

También puedes usar el método `restore` en una consulta:

	User::withTrashed()->where('account_id', 1)->restore();

Así como el método `withTrashed`, el método `restore` puede ser usado en tus relaciones:

	$user->posts()->restore();

Si deseas realmente eliminar registros de la base de datos, puedes usar el método `forceDelete`:

	$user->forceDelete();

El método `forceDelete` también funciona en relaciones:

	$user->posts()->forceDelete();

Para determinar si la instancia de un modelo ha sido eliminado flexiblemente, puedes usar el método `trashed`:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## Marcas de tiempo

De forma predeterminada, Eloquent mantendrá las columnas `created_at` y `updated_at` en tu base de datos actualizadas automáticamente. Simplemente agrega esas columnas de tipo `timestamp` a tu tabla y Eloquent hará el resto. Si no deseas que Eloquent mantenga esas columnas, agrega la siguiente propiedad tu modelo:

#### Deshabilitando marcas de tiempo automáticas

	class User extends Eloquent {

		protected $table = 'users';

		public $timestamps = false;

	}

Si deseas personalizar el formato de tus marcas de tiempo, puedes sobreescribir el método `getDateFormat` en tu modelo:

#### Definiendo un formato de marca de tiempo personalizada

	class User extends Eloquent {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## Consultas con alcance

Las consultas con alcance te permiten reusar fácilmente la lógia de tus modelos. Para definir una consulta con alcance, simplemente define un método en el modelo con el prefijo `scope`:

#### Definiendo una consulta con alcance

	class User extends Eloquent {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

		public function scopeWomen($query)
		{
			return $query->whereGender('W');
		}

	}

#### Utilizando una consulta con alcance

	$users = User::popular()->women()->orderBy('created_at')->get();

#### Alcances dinámicos

Algunas veces puedes desear definir un alcance que acepta parámetros. Simplemente agrega tus parámetros a la función de alcance:

	class User extends Eloquent {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

Luego pasa el parámetro al llamado del alcance:

	$users = User::ofType('member')->get();

<a name="relationships"></a>
## Relaciones

Por supuesto, tus tablas de la base de datos probablemente se relacionen una con otras. Por ejemplo, un artículo en un blog tiene muchos comentarios, o un recibo puede relacionarse con el usuario que lo solicitó. Eloquent hace el manejo y el funcionamiento de esas relaciones fácilmente. Laravel soporta varios tipos de relaciones:

- [Uno a uno](#one-to-one)
- [Uno a muchos](#one-to-many)
- [Muchos a muchos](#many-to-many)
- [Muchos a través](#has-many-through)
- [Relaciones polimórficas](#polymorphic-relations)
- [Relaciones polimórficas muchos a muchos](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### Uno a uno

Una relación una a una es una relación muy básica. Por ejemplo un modelo `User` puede tener un `Phone`. Podemos definir está relación en Eloquent:

#### Definiendo una relación una a una

	class User extends Eloquent {

		public function phone()
		{
			return $this->hasOne('Phone');
		}

	}

El primer argumento pasado al método `hasOne` es el nombre del modelo relacionado. Una vez la relación esté definida podemos obtenerla usando las [propiedades dinámicas](#dynamic-properties) de Eloquent:

	$phone = User::find(1)->phone;

La consulta SQL ejecutada por esta sentencia es:

	select * from users where id = 1

	select * from phones where user_id = 1

Es de notar que Eloquent asume el nombre de la llave foránea basado en el nombre del modelo. En este ejemplo, el modelo `Phone` asume usar la llave foránea `user_id`. Si deseas cambiar está convención, puedes pasar un segundo parámetro al método `hasOne`. Además, puedes pasar un tercer parámetro al método para especificar que columna local debe ser usada para la asociación:

	return $this->hasOne('Phone', 'foreign_key');

	return $this->hasOne('Phone', 'foreign_key', 'local_key');

#### Definiendo el inverso de una relación

Para definir el inverso de una relación en el modelo `Phone`, usamos el método `belongsTo`:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

En el ejemplo anterior, Eloquent buscará la columna `user_id` en la tabla `phones`. Si quieres definir otra columna como llave foránea, puedes pasar un segundo parámetro al método `belongsTo`:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User', 'local_key');
		}

	}

Adicionalmente, puedes pasar un tercer parámetro para especificar el nombre de la columna asociada a la tabla padre:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User', 'local_key', 'parent_key');
		}

	}

<a name="one-to-many"></a>
### Uno a muchos

Un ejemplo de una relación uno a muchos es un artículo en un blog "tiene muchos" comentarios. Podemos modelar esta relación así:

	class Post extends Eloquent {

		public function comments()
		{
			return $this->hasMany('Comment');
		}

	}

Ahora podemos acceder a los comentarios del artículo a través de [propiedades dinámicas](#dynamic-properties):

	$comments = Post::find(1)->comments;

Si necesitas agregar más condiciones a los comentarios obtenidos, puedes llamar al método `comments` y seguir encadenando condiciones:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

De nuevo, puedes sobreescribir la convención de las llaves foráneas pasando un segundo argumento al método `hasMany`. Y, como en la relación `hasOne`, la columna local también se puede especificar:

	return $this->hasMany('Comment', 'foreign_key');

	return $this->hasMany('Comment', 'foreign_key', 'local_key');

Para definir el inverso de la relación, en el modelo `Comment`, usamos el método `belongsTo`:

#### Definiendo el inverso de una relación

	class Comment extends Eloquent {

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

<a name="many-to-many"></a>
### Muchos a muchos

Many-to-many relations are a more complicated relationship type. An example of such a relationship is a user with many roles, where the roles are also shared by other users. For example, many users may have the role of "Admin". Three database tables are needed for this relationship: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names, and should have `user_id` and `role_id` columns.

We can define a many-to-many relation using the `belongsToMany` method:

	class User extends Eloquent {

		public function roles()
		{
			return $this->belongsToMany('Role');
		}

	}

Now, we can retrieve the roles through the `User` model:

	$roles = User::find(1)->roles;

If you would like to use an unconventional table name for your pivot table, you may pass it as the second argument to the `belongsToMany` method:

	return $this->belongsToMany('Role', 'user_roles');

You may also override the conventional associated keys:

	return $this->belongsToMany('Role', 'user_roles', 'user_id', 'foo_id');

Of course, you may also define the inverse of the relationship on the `Role` model:

	class Role extends Eloquent {

		public function users()
		{
			return $this->belongsToMany('User');
		}

	}

<a name="has-many-through"></a>
### Muchos a través

The "has many through" relation provides a convenient short-cut for accessing distant relations via an intermediate relation. For example, a `Country` model might have many `Posts` through a `Users` model. The tables for this relationship would look like this:

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

Even though the `posts` table does not contain a `country_id` column, the `hasManyThrough` relation will allow us to access a country's posts via `$country->posts`. Let's define the relationship:

	class Country extends Eloquent {

		public function posts()
		{
			return $this->hasManyThrough('Post', 'User');
		}

	}

If you would like to manually specify the keys of the relationship, you may pass them as the third and fourth arguments to the method:

	class Country extends Eloquent {

		public function posts()
		{
			return $this->hasManyThrough('Post', 'User', 'country_id', 'user_id');
		}

	}

<a name="polymorphic-relations"></a>
### Relaciones polimórficas

Polymorphic relations allow a model to belong to more than one other model, on a single association. For example, you might have a photo model that belongs to either a staff model or an order model. We would define this relation like so:

	class Photo extends Eloquent {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

	class Order extends Eloquent {

		public function photos()
		{
			return $this->morphMany('Photo', 'imageable');
		}

	}

Now, we can retrieve the photos for either a staff member or an order:

#### Retrieving A Polymorphic Relation

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

#### Retrieving The Owner Of A Polymorphic Relation

However, the true "polymorphic" magic is when you access the staff or order from the `Photo` model:

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

The `imageable` relation on the `Photo` model will return either a `Staff` or `Order` instance, depending on which type of model owns the photo.

To help understand how this works, let's explore the database structure for a polymorphic relation:

#### Polymorphic Relation Table Structure

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

The key fields to notice here are the `imageable_id` and `imageable_type` on the `photos` table. The ID will contain the ID value of, in this example, the owning staff or order, while the type will contain the class name of the owning model. This is what allows the ORM to determine which type of owning model to return when accessing the `imageable` relation.

<a name="many-to-many-polymorphic-relations"></a>
### Relaciones Polimórficas muchos a muchos

In addition to traditional polymorphic relations, you may also specify many-to-many polymorphic relations. For example, a blog `Post` and `Video` model could share a polymorphic relation to a `Tag` model. First, let's examine the table structure:

#### Polymorphic Many To Many Relation Table Structure

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

Next, we're ready to setup the relationships on the model. The `Post` and `Video` model will both have a `morphToMany` relationship via a `tags` method:

	class Post extends Eloquent {

		public function tags()
		{
			return $this->morphToMany('Tag', 'taggable');
		}

	}

The `Tag` model may define a method for each of its relationships:

	class Tag extends Eloquent {

		public function posts()
		{
			return $this->morphedByMany('Post', 'taggable');
		}

		public function videos()
		{
			return $this->morphedByMany('Video', 'taggable');
		}

	}

<a name="querying-relations"></a>
## Consultas sobre relaciones

When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, you wish to pull all blog posts that have at least one comment. To do so, you may use the `has` method:

#### Querying Relations When Selecting

	$posts = Post::has('comments')->get();

You may also specify an operator and a count:

	$posts = Post::has('comments', '>=', 3)->get();

If you need even more power, you may use the `whereHas` and `orWhereHas` methods to put "where" conditions on your `has` queries:

	$posts = Post::whereHas('comments', function($q)
	{
		$q->where('content', 'like', 'foo%');

	})->get();

<a name="dynamic-properties"></a>
### Propiedades dinámicas

Eloquent allows you to access your relations via dynamic properties. Eloquent will automatically load the relationship for you, and is even smart enough to know whether to call the `get` (for one-to-many relationships) or `first` (for one-to-one relationships) method.  It will then be accessible via a dynamic property by the same name as the relation. For example, with the following model `$phone`:

	class Phone extends Eloquent {

		public function user()
		{
			return $this->belongsTo('User');
		}

	}

	$phone = Phone::find(1);

Instead of echoing the user's email like this:

	echo $phone->user()->first()->email;

It may be shortened to simply:

	echo $phone->user->email;

> **Note:** Relationships that return many results will return an instance of the `Illuminate\Database\Eloquent\Collection` class.

<a name="eager-loading"></a>
## Carga impaciente

Eager loading exists to alleviate the N + 1 query problem. For example, consider a `Book` model that is related to `Author`. The relationship is defined like so:

	class Book extends Eloquent {

		public function author()
		{
			return $this->belongsTo('Author');
		}

	}

Now, consider the following code:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries.

Thankfully, we can use eager loading to drastically reduce the number of queries. The relationships that should be eager loaded may be specified via the `with` method:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

In the loop above, only two queries will be executed:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

Wise use of eager loading can drastically increase the performance of your application.

Of course, you may eager load multiple relationships at one time:

	$books = Book::with('author', 'publisher')->get();

You may even eager load nested relationships:

	$books = Book::with('author.contacts')->get();

In the example above, the `author` relationship will be eager loaded, and the author's `contacts` relation will also be loaded.

### Restricciones en precargas

Sometimes you may wish to eager load a relationship, but also specify a condition for the eager load. Here's an example:

	$users = User::with(array('posts' => function($query)
	{
		$query->where('title', 'like', '%first%');

	}))->get();

In this example, we're eager loading the user's posts, but only if the post's title column contains the word "first".

Of course, eager loading Closures aren't limited to "constraints". You may also apply orders:

	$users = User::with(array('posts' => function($query)
	{
		$query->orderBy('created_at', 'desc');

	}))->get();

### Carga impaciente

It is also possible to eagerly load related models directly from an already existing model collection. This may be useful when dynamically deciding whether to load related models or not, or in combination with caching.

	$books = Book::all();

	$books->load('author', 'publisher');

<a name="inserting-related-models"></a>
## Insertando modelos relacionados

You will often need to insert new related models. For example, you may wish to insert a new comment for a post. Instead of manually setting the `post_id` foreign key on the model, you may insert the new comment from its parent `Post` model directly:

#### Attaching A Related Model

	$comment = new Comment(array('message' => 'A new comment.'));

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

In this example, the `post_id` field will automatically be set on the inserted comment.

### Asociando modelos (Pertenece a)

When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### Insertados modelos relacionados (Muchos a muchos)

You may also insert related models when working with many-to-many relations. Let's continue using our `User` and `Role` models as examples. We can easily attach new roles to a user using the `attach` method:

#### Attaching Many To Many Models

	$user = User::find(1);

	$user->roles()->attach(1);

You may also pass an array of attributes that should be stored on the pivot table for the relation:

	$user->roles()->attach(1, array('expires' => $expires));

Of course, the opposite of `attach` is `detach`:

	$user->roles()->detach(1);

You may also use the `sync` method to attach related models. The `sync` method accepts an array of IDs to place on the pivot table. After this operation is complete, only the IDs in the array will be on the intermediate table for the model:

#### Using Sync To Attach Many To Many Models

	$user->roles()->sync(array(1, 2, 3));

You may also associate other pivot table values with the given IDs:

#### Adding Pivot Data When Syncing

	$user->roles()->sync(array(1 => array('expires' => true)));

Sometimes you may wish to create a new related model and attach it in a single command. For this operation, you may use the `save` method:

	$role = new Role(array('name' => 'Editor'));

	User::find(1)->roles()->save($role);

In this example, the new `Role` model will be saved and attached to the user model. You may also pass an array of attributes to place on the joining table for this operation:

	User::find(1)->roles()->save($role, array('expires' => $expires));

<a name="touching-parent-timestamps"></a>
## Actualizando marcas de tiempo de relaciones padre

When a model `belongsTo` another model, such as a `Comment` which belongs to a `Post`, it is often helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically touch the `updated_at` timestamp of the owning `Post`. Eloquent makes it easy. Just add a `touches` property containing the names of the relationships to the child model:

	class Comment extends Eloquent {

		protected $touches = array('post');

		public function post()
		{
			return $this->belongsTo('Post');
		}

	}

Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated:

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## Trabajando con tablas intermedias

As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the `pivot` table on the models:

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used as any other Eloquent model.

By default, only the keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

	return $this->belongsToMany('Role')->withPivot('foo', 'bar');

Now the `foo` and `bar` attributes will be accessible on our `pivot` object for the `Role` model.

If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

	return $this->belongsToMany('Role')->withTimestamps();

To delete all records on the pivot table for a model, you may use the `detach` method:

#### Deleting Records On A Pivot Table

	User::find(1)->roles()->detach();

Note that this operation does not delete records from the `roles` table, but only from the pivot table.

#### Defining A Custom Pivot Model

Laravel also allows you to define a custom Pivot model. To define a custom model, first create your own "Base" model class that extends `Eloquent`. In your other Eloquent models, extend this custom base model instead of the default `Eloquent` base. In your base model, add the following function that returns an instance of your custom Pivot model:

	public function newPivot(Model $parent, array $attributes, $table, $exists)
	{
		return new YourCustomPivot($parent, $attributes, $table, $exists);
	}

<a name="collections"></a>
## Colecciones

All multi-result sets returned by Eloquent, either via the `get` method or a `relationship`, will return a collection object. This object implements the `IteratorAggregate` PHP interface so it can be iterated over like an array. However, this object also has a variety of other helpful methods for working with result sets.

For example, we may determine if a result set contains a given primary key using the `contains` method:

#### Checking If A Collection Contains A Key

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Collections may also be converted to an array or JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

If a collection is cast to a string, it will be returned as JSON:

	$roles = (string) User::find(1)->roles;

Eloquent collections also contain a few helpful methods for looping and filtering the items they contain:

#### Iterating Collections

	$roles = $user->roles->each(function($role)
	{
		//
	});

#### Filtering Collections

When filtering collections, the callback provided will be used as callback for [array_filter](http://php.net/manual/en/function.array-filter.php).

	$users = $users->filter(function($user)
	{
		return $user->isAdmin();
	});

> **Note:** When filtering a collection and converting it to JSON, try calling the `values` function first to reset the array's keys.

#### Applying A Callback To Each Collection Object

	$roles = User::find(1)->roles;

	$roles->each(function($role)
	{
		//
	});

#### Sorting A Collection By A Value

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});

#### Sorting A Collection By A Value

	$roles = $roles->sortBy('created_at');

Sometimes, you may wish to return a custom Collection object with your own added methods. You may specify this on your Eloquent model by overriding the `newCollection` method:

#### Returning A Custom Collection Type

	class User extends Eloquent {

		public function newCollection(array $models = array())
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Descriptores de acceso y Mutadores

Eloquent provides a convenient way to transform your model attributes when getting or setting them. Simply define a `getFooAttribute` method on your model to declare an accessor. Keep in mind that the methods should follow camel-casing, even though your database columns are snake-case:

#### Defining An Accessor

	class User extends Eloquent {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

In the example above, the `first_name` column has an accessor. Note that the value of the attribute is passed to the accessor.

Mutators are declared in a similar fashion:

#### Defining A Mutator

	class User extends Eloquent {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Mutadores de fecha

By default, Eloquent will convert the `created_at`, `updated_at`, and `deleted_at` columns to instances of [Carbon](https://github.com/briannesbitt/Carbon), which provides an assortment of helpful methods, and extends the native PHP `DateTime` class.

You may customize which fields are automatically mutated, and even completely disable this mutation, by overriding the `getDates` method of the model:

	public function getDates()
	{
		return array('created_at');
	}

When a column is considered a date, you may set its value to a UNIX timetamp, date string (`Y-m-d`), date-time string, and of course a `DateTime` / `Carbon` instance.

To totally disable date mutations, simply return an empty array from the `getDates` method:

	public function getDates()
	{
		return array();
	}

<a name="model-events"></a>
## Eventos de modelos

Eloquent models fire several events, allowing you to hook into various points in the model's lifecycle using the following methods: `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.

Whenever a new item is saved for the first time, the `creating` and `created` events will fire. If an item is not new and the `save` method is called, the `updating` / `updated` events will fire. In both cases, the `saving` / `saved` events will fire.

If `false` is returned from the `creating`, `updating`, `saving`, or `deleting` events, the action will be cancelled:

#### Cancelling Save Operations Via Events

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

Eloquent models also contain a static `boot` method, which may provide a convenient place to register your event bindings.

#### Setting A Model Boot Method

	class User extends Eloquent {

		public static function boot()
		{
			parent::boot();

			// Setup event bindings...
		}

	}

<a name="model-observers"></a>
## Observadores de modelos

To consolidate the handling of model events, you may register a model observer. An observer class may have methods that correspond to the various model events. For example, `creating`, `updating`, `saving` methods may be on an observer, in addition to any other model event name.

So, for example, a model observer might look like this:

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

You may register an observer instance using the `observe` method:

	User::observe(new UserObserver);

<a name="converting-to-arrays-or-json"></a>
## Convirtiendo a Arreglos / JSON

When building JSON APIs, you may often need to convert your models and relationships to arrays or JSON. So, Eloquent includes methods for doing so. To convert a model and its loaded relationship to an array, you may use the `toArray` method:

#### Converting A Model To An Array

	$user = User::with('roles')->first();

	return $user->toArray();

Note that entire collections of models may also be converted to arrays:

	return User::all()->toArray();

To convert a model to JSON, you may use the `toJson` method:

#### Converting A Model To JSON

	return User::find(1)->toJson();

Note that when a model or collection is cast to a string, it will be converted to JSON, meaning you can return Eloquent objects directly from your application's routes!

#### Returning A Model From A Route

	Route::get('users', function()
	{
		return User::all();
	});

Sometimes you may wish to limit the attributes that are included in your model's array or JSON form, such as passwords. To do so, add a `hidden` property definition to your model:

#### Hiding Attributes From Array Or JSON Conversion

	class User extends Eloquent {

		protected $hidden = array('password');

	}

> **Note:** When hiding relationships, use the relationship's **method** name, not the dynamic accessor name.

Alternatively, you may use the `visible` property to define a white-list:

	protected $visible = array('first_name', 'last_name');

<a name="array-appends"></a>
Occasionally, you may need to add array attributes that do not have a corresponding column in your database. To do so, simply define an accessor for the value:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

Once you have created the accessor, just add the value to the `appends` property on the model:

	protected $appends = array('is_admin');

Once the attribute has been added to the `appends` list, it will be included in both the model's array and JSON forms.
