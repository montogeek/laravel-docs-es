# Controladores

- [Controladores básicos](#basic-controllers)
- [Controladores con filtros](#controller-filters)
- [Controladores REST](#restful-controllers)
- [Controladores de recursos](#resource-controllers)
- [Manejando métodos faltantes](#handling-missing-methods)

<a name="basic-controllers"></a>
## Controladores básicos

En vez de definir toda tu lógica de rutas en un solo archivo `routes.php`, probablemente preferirías organizar este comportamiento con las clases de los Controladores. Los Controladores pueden agrupar lógica de rutas relacionadas en una clase, así como tomar ventaja de características más avanzadas del framework como la [inyección de dependencias](/docs/ioc) automática.

Los Controladores se guardan normalmente en el directorio `app/controllers`, este directorio esta registrado de forma predeterminada en la opción `classmap` de tu archivo `composer.json`. Sin embargo, los controladores pueden guardarse en cualquier directorio o sub-directorio. La declaración de Rutas no dependen del lugar en disco donde se encuentre la clase del controlador. Así, mientras Composer conozca como autocargar la clase del controlador, este se puede guardar en el lugar que quieras.

Un ejemplo de una clase básica de un controlador.

	class UserController extends BaseController {

		/**
		 * Show the profile for the given user.
		 */
		public function showProfile($id)
		{
			$user = User::find($id);

			return View::make('user.profile', array('user' => $user));
		}

	}

Todos los controladores deben extender de la clase `BaseController`. La clase `BaseController` también se guarda en el directorio `app/controllers`, y puede ser usada para almacenar lógica común a todos los controladores. La clase `BaseController` extiende de la clase base `Controller` del framework. Ahora, podemos crear una ruta a la acción de este controlador así:

	Route::get('user/{id}', 'UserController@showProfile');

Si elegiste organizar o poner en una jerarquía tus controladores usando los namespaces de PHP, simplemente usa el nombre de la clase completo cuando definas la ruta:

	Route::get('foo', 'Namespace\FooController@method');

> **Nota:** Ya que estamos usando [Composer](http://getcomposer.org) para autocargar nuestras clases de PHP, los controladores pueden estar en cualquier lugar el sistema de archivos mientras Composer sepa como cargarlos. El directorio `controller` no obliga a usar una estructura de carpeta estricta en tu aplicación. Crear rutas para los controladores esta completamente separado del sistema de archivos.

Puedes especificar nombres a tus rutas de controladores:

	Route::get('foo', array('uses' => 'FooController@method',
											'as' => 'name'));

Para generar una URL a la acción de un controlador, puedes usar el método `URL::action` o el método de ayuda `helper`:

	$url = URL::action('FooController@method');

	$url = action('FooController@method');

Puedes acceder al nombre de la acción del controlador que se está ejecutando usando el método `currentRouteAction`

	$action = Route::currentRouteAction();

<a name="controller-filters"></a>
## Controladores con filtros

Los [Filtros](/docs/routing#route-filters) puedes ser especificados en las rutas de controladores, muy similar a las rutas regulares:

	Route::get('profile', array('before' => 'auth',
				'uses' => 'UserController@showProfile'));

Sin embargo, también puedes especificar filtros desde tu controlador:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('auth', array('except' => 'getLogin'));

			$this->beforeFilter('csrf', array('on' => 'post'));

			$this->afterFilter('log', array('only' =>
								array('fooAction', 'barAction')));
		}

	}

Puedes especificar filtros de controladores en la línea usando una Clausura:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter(function()
			{
				//
			});
		}

	}

Si quieres usar otro método del controlador como filtro, puedes hacerlo usando la síntaxis `@` para definir el filtro:

	class UserController extends BaseController {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->beforeFilter('@filterRequests');
		}

		/**
		 * Filter the incoming requests.
		 */
		public function filterRequests($route, $request)
		{
			//
		}

	}

<a name="restful-controllers"></a>
## Controladores REST

Laravel te permite definir fácilmente en una sola ruta el manejo de cada acción en un controlador usando simples convenciones de nombres REST. Primero, define la ruta usando el método `Route::controller`:

#### Definiendo un controlador RESTful

	Route::controller('users', 'UserController');

El método `controller` acepta dos argumentos. El primero es la URI base del controlador, mientras el segundo es el nombre de la clase del controlador. A continuación, simplemente crea métodos en tu controlador, con un prefijo que corresponda al verbo HTTP al cual responderán:

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

	}

El método `index` responderá a la URI base definida para el controlador, que en este caso es `users`.

Si la acción en tu controlador tiene varias palabras, puedes acceder a la acción usando la síntaxis `guión` en la URI. Por ejemplo, la siguiente acción en nuestro controlador `UserController` responderá a la URI `users/admin-profile`:

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## Controladores de recursos

Los controladores de recursos hacen más fácil la creación de controladores REST para recursos. Por ejemplo, es posible que desees crear un controlador que maneje 'photos' guardadas en tu aplicación. Usando el comando `controller:make` disponible a través de Artisan y el método `Route::resource`, podemos rápidamente crear dicho controlador.

Para crear el controlador a través de la línea de comando, ejecuta el siguiente comando:

	php artisan controller:make PhotoController

Ahora podemos registrar una ruta de recursos para el controlador:

	Route::resource('photo', 'PhotoController');

Está única declaración de ruta crea múltiples rutas para manejar una variedad de acciones REST para el recurso `photo`. Del mismo modo, el controlador generado ya tendrá los métodos para cada una de estas acciones con notas informando cuales URIs y verbos HTTP manejará.

#### Acciones manejadas por un controlador de recursos

Verbo     | Dirección                   | Acción       | Nombre de la ruta
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

Algunas veces puedes solamente necesitar manejar un subconjunto de las acciones del recurso:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

Y, puedes también especificar un subconjunto de acciones para manejar en la ruta:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

	Route::resource('photo', 'PhotoController',
					array('except' => array('create', 'store', 'update', 'destroy')));

De forma predeterminada, todas las acciones de un controlador de recursos tienen una ruta con nombre, sin embargo, puedes sobreescribir estos nombres pasando el arreglo `names` a las opciones de la ruta:

	Route::resource('photo', 'PhotoController',
					array('names' => array('create' => 'photo.build')));

<a name="handling-missing-methods"></a>
## Manejando métodos faltantes

Un método comodín puede ser definido para ser llamado cuando ningún otro método concuerde en el controlador dado. El método debe llamarse `missingMethod` y recibe el método y un arreglo de los parámetros de la petición:

#### Defining A Catch-All Method

	public function missingMethod($parameters = array())
	{
		//
	}
