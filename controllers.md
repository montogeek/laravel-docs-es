# Controllers

- [Controladores básicos](#basic-controllers)
- [Controladores con filtros](#controller-filters)
- [Controladores REST](#restful-controllers)
- [Controladores de recursos](#resource-controllers)
- [Manejando métodos faltantes](#handling-missing-methods)

<a name="basic-controllers"></a>
## Controladores básicos

En vez de definir toda tu lógica de rutas en un solo archivo `routes.php`, puedes querer organizar esta comportamiento usando las clases de los Controladores. Los Controladores pueden agrupar lógica de rutas relacionadas en una clase, así como tomar ventaja de características más avanzadas del framework como la [inyección de dependencias](/docs/ioc) automática.

Los Controladores se guardan normalmente en el directorio `app/controllers`, este directorio esta registrado de forma predeterminada en la opción `classmap` de tu archivo `composer.json`. Sin embargo, los controladores pueden guardarse en cualquier directorio o sub-directorio. La declaración de Rutas no dependen del lugar de la clase del controlador en el disco. Así, mientras Composer conozco como autocargar la clase del controlador, este se puede guardar en el lugar que quieras.

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

Todos los controladores deberían extender la clase `BaseController`. La clase `BaseController` también se guarda en el directorio `app/controllers`, y puede ser usada para almacenar lógica común a todos los controladores. La clase `BaseController` extiende la clase base `Controller` del framework. Ahora, podemos crear una ruta a la acción de este controlador así:

	Route::get('user/{id}', 'UserController@showProfile');

Si elegiste organizar o poner en una jerarquía tus controladores usando los namespaces de PHP, simplemente usa el nombre de la clase completo cuando definas la ruta:

	Route::get('foo', 'Namespace\FooController@method');

> **Nota:** Ya que estamos usando [Composer](http://getcomposer.org) para autocargar nuestras clases de PHP, los controladores pueden estar en cualquier lugar el sistema de archivos mientras Composer sepa como cargarlos. El directorio `controller` no obliga a usar una estructura estricta en tu aplicación. Crear rutas para los controladores esta completamente separado del sistema de archivos.

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

Sin embargo, puedes también especificar filtros desde tu controlador:

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

Puedes especificar filtros de controladores en línea usando una Clausura:

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

Laravel allows you to easily define a single route to handle every action in a controller using simple, REST naming conventions. First, define the route using the `Route::controller` method:

#### Defining A RESTful Controller

	Route::controller('users', 'UserController');

The `controller` method accepts two arguments. The first is the base URI the controller handles, while the second is the class name of the controller. Next, just add methods to your controller, prefixed with the HTTP verb they respond to:

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

The `index` methods will respond to the root URI handled by the controller, which, in this case, is `users`.

If your controller action contains multiple words, you may access the action using "dash" syntax in the URI. For example, the following controller action on our `UserController` would respond to the `users/admin-profile` URI:

	public function getAdminProfile() {}

<a name="resource-controllers"></a>
## Controladores de recursos

Resource controllers make it easier to build RESTful controllers around resources. For example, you may wish to create a controller that manages "photos" stored by your application. Using the `controller:make` command via the Artisan CLI and the `Route::resource` method, we can quickly create such a controller.

To create the controller via the command line, execute the following command:

	php artisan controller:make PhotoController

Now we can register a resourceful route to the controller:

	Route::resource('photo', 'PhotoController');

This single route declaration creates multiple routes to handle a variety of RESTful actions on the photo resource. Likewise, the generated controller will already have stubbed methods for each of these actions with notes informing you which URIs and verbs they handle.

#### Actions Handled By Resource Controller

Verb      | Path                        | Action       | Route Name
----------|-----------------------------|--------------|---------------------
GET       | /resource                   | index        | resource.index
GET       | /resource/create            | create       | resource.create
POST      | /resource                   | store        | resource.store
GET       | /resource/{resource}        | show         | resource.show
GET       | /resource/{resource}/edit   | edit         | resource.edit
PUT/PATCH | /resource/{resource}        | update       | resource.update
DELETE    | /resource/{resource}        | destroy      | resource.destroy

Sometimes you may only need to handle a subset of the resource actions:

	php artisan controller:make PhotoController --only=index,show

	php artisan controller:make PhotoController --except=index

And, you may also specify a subset of actions to handle on the route:

	Route::resource('photo', 'PhotoController',
					array('only' => array('index', 'show')));

	Route::resource('photo', 'PhotoController',
					array('except' => array('create', 'store', 'update', 'destroy')));

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

	Route::resource('photo', 'PhotoController',
					array('names' => array('create' => 'photo.build')));

<a name="handling-missing-methods"></a>
## Manejando métodos faltantes

A catch-all method may be defined which will be called when no other matching method is found on a given controller. The method should be named `missingMethod`, and receives the method and parameter array for the request:

#### Defining A Catch-All Method

	public function missingMethod($parameters = array())
	{
		//
	}
