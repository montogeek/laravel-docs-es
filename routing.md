# Routing

- [Rutas báscias](#basic-routing)
- [Rutas con parámetros](#route-parameters)
- [Rutas con filtros](#route-filters)
- [Rutas con nombre](#named-routes)
- [Grupos de rutas](#route-groups)
- [Rutas con subdominios](#sub-domain-routing)
- [Rutas con prefijos](#route-prefixing)
- [Rutas con modelos de Eloquent](#route-model-binding)
- [Generando errores 404](#throwing-404-errors)
- [Rutas con controladores](#routing-to-controllers)

<a name="basic-routing"></a>
## Rutas báscias

La mayoría de las rutas para tu aplicación serán definidias en el archivo `app/routes.php`. La definición de una ruta de Laravel más simple consiste en una URI y una retrollamada a una Clausura. <!-- TODO: Referencia Wikipedia y foot notes-->

#### Ruta GET básica

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### Ruta POST básica

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

#### Registrando una ruta para varios verbos HTTP

	Route::match(array('GET', 'POST'), '/', function()
	{
		return 'Hello World';
	});

#### Registrando una ruta para cualquier verbo HTTP

	Route::any('foo', function()
	{
		return 'Hello World';
	});

#### Obligar una ruta para responser solo por HTTPS

	Route::get('foo', array('https', function()
	{
		return 'Must be over HTTPS';
	}));

Con frecuencia, necesitará generar URLs para tus rutas, lo puedes hacer usando el método: `URL::to`.

	$url = URL::to('foo');

<a name="route-parameters"></a>
## Rutas con parámetros

	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

#### Rutas con parámetros opcionales

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

#### Rutas con parámetros opcionales y valores predeterminados

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

#### Rutas con expresiones regulares como restricciones

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

#### Múltiples expresiones regulares como restricciones

Por supuesto, puedes pasar un arreglo de restricciones cuando sea necesario:

	Route::get('user/{id}/{name}', function($id, $name)
	{
		//
	})
	->where(array('id' => '[0-9]+', 'name' => '[a-z]+'))

#### Definición de patrones globales

Si deseas que el parámetro de una ruta siempre sea restrictiva por una expresión regular, puedes usar el método `pattern`:

	Route::pattern('id', '[0-9]+');

	Route::get('user/{id}', function($id)
	{
		// Only called if {id} is numeric.
	});

#### Obtener el valor del parámetro de una ruta

Si necesitas obtener el valor de algún parámetro de una ruta fuera de la ruta, puedes usar el método `Route::input`:

	Route::filter('foo', function()
	{
		if (Route::input('id') == 1)
		{
			//
		}
	});

<a name="route-filters"></a>
## Rutas con filtros

Filtros en rutas proveen una manera conveniente para limitar el acceso a la ruta dada, muy útil para definir áreas en tu aplicación que requieren autentificación.
Laravel incluye varios filtros: filtro `auth`, filtro `auth.basic`, filtro `guest`, filtro `csrf`, los cuales están definidos en el archivo `app/filters.php`.

#### Definiendo un filtro para una ruta

	Route::filter('old', function()
	{
		if (Input::get('age') < 200)
		{
			return Redirect::to('home');
		}
	});

Si el filtro retorna una respuesta, esa respuesta será considerada la respuesta de la petición y la ruta no será ejecutada. Cualquier filtro `después` de la ruta tampoco serán ejecutados.

#### Agregar un filtro a una ruta

	Route::get('user', array('before' => 'old', function()
	{
		return 'You are over 200 years old!';
	}));

#### Agregar un filtro a una acción de un controlador

	Route::get('user', array('before' => 'old', 'uses' => 'UserController@showProfile'));

#### Agregar varios filtros a una ruta

	Route::get('user', array('before' => 'auth|old', function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### Agregar varios filtros a una ruta a través de un arreglo

	Route::get('user', array('before' => array('auth', 'old'), function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

#### Especificando parámetros en un filtro

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('user', array('before' => 'age:200', function()
	{
		return 'Hello World';
	}));

Filtros que se ejecutan después de una ruta tienen disponible un `$response` como tercer parámetro:

	Route::filter('log', function($route, $request, $response)
	{
		//
	});

#### Filtros basados en patrones

Puedes también espeficiar que un filtro se ejecute a un conjunto completo de rutas basado en sus URIs.

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

En el ejemplo anterior, el filtro `admin` se aplicará a todas las rutas que empiezen con `admin/`. El asterisco es usado como un comodín y coincidará cualquier combinación de carácteres.

También puedes restringir la ejecución de filtros basados en patrones por verbos HTTPs.

	Route::when('admin/*', 'admin', array('post'));

#### Filtros con clases

Para filtros avanzados, tal vez desees usar una clase en vez de una Clausura.
Como las clases para filtros son resueltas fuera del [contenedor IoC](/docs/ioc) de la aplicación, podrás utilizar inyección de dependencias en estos filtros para mejorar la capacidad de pruebas.

#### Registrando una clase para un filtro

	Route::filter('foo', 'FooFilter');

De forma predeterminada, el método `filter` en la clase `FooFilter` será ejecutado:

	class FooFilter {

		public function filter()
		{
			// Filter logic...
		}

	}

Si no desea usar el método `filter`, simplemente específica otro:

	Route::filter('foo', 'FooFilter@foo');

<a name="named-routes"></a>
## Rutas con nombre

Rutas con nombre hacen que la referencia a rutas cuando se generan redirecciones o URLs sea mucho más conveniente.

	Route::get('user/profile', array('as' => 'profile', function()
	{
		//
	}));

También puedes especificar nombres de rutas para acciones en controladores

	Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

Ahora, puedes usar el nombre de la ruta cuando generes URLs o redirecciones:

	$url = URL::route('profile');

	$redirect = Redirect::route('profile');

Puedes acceder al nombre de una ruta que se está ejecutando a través del método `currentRouteName`

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## Grupos de rutas

Algunas veces podrías necesitar aplicar un filtro a un grupo de rutas. En vez de especificarlos para cada ruta, puedes hacer para un grupo de rutas:

	Route::group(array('before' => 'auth'), function()
	{
		Route::get('/', function()
		{
			// Has Auth Filter
		});

		Route::get('user/profile', function()
		{
			// Has Auth Filter
		});
	});

Puedes usar el parámetro `namespace` en el arreglo de la definición del grupo de rutas para especificar que todos los controladores dentro del grupo pertenecen al mismo namespace<!-- TODO: Referencia Wikipedia y foot notes-->:

	Route::group(array('namespace' => 'Admin'), function()
	{
		//
	});

<a name="sub-domain-routing"></a>
## Rutas con subdominios

Las rutas en Laravel son capaces de manejar comódines en sub-dominios y definir parámetros de comódines desde el dominio principal:

#### Registrando rutas de subdominios

	Route::group(array('domain' => '{account}.myapp.com'), function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});

<a name="route-prefixing"></a>
## Rutas con prefijos

Un grupo de rutas pueden usar la opción `prefix` en el arreglo de sus atributos para definir un prefijo a todo el grupo:

#### Grupo de rutas con prefijo

	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## Rutas con modelos de Eloquent

Vincular modelos provee una manera conveniente para inyectar instancias del modelo en las rutas. Por ejemplo, en vez de inyectar el ID de un usuario, puedes inyectar el modelo Usuario completo que concuerde con un ID dado. Primero, usa el método `Route::model()` para especificar el modelo que será usado para un parámetro dado:

#### Vinculando un parámetro a un modelo

	Route::model('user', 'User');

A continuación, defina una ruta que contenga el parámetro `{user}`:

	Route::get('profile/{user}', function(User $user)
	{
		//
	});

Como hemos vinculado el parámetro `{user}` al modelo `User`, una instancia `User` será inyectado a la ruta. Así, por ejemplo, una petición a `profile/1` inyectará una instancia con el ID igual a 1.

> **Note:** Si una instancia del modelo no se encuentra en la base de datos, se ejecutará un error 404.

Si deseas especificar tu propia implementación de un modelo no encontrado, puedes pasar una Clausura como tercer parámetro del método `model`:

	Route::model('user', 'User', function()
	{
		throw new NotFoundException;
	});

Algunas veces deseas usar tu propia forma de resolver los parámetros de las rutas. Simplemente usa el método `Route::bing`:

	Route::bind('user', function($value, $route)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## Generando errores 404

Existen dos formas de ejecutar un error 404 desde una ruta. Primero, puedes usar el método `App::abort`:

	App::abort(404);

Segundo, ejecutar una instancia de `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.

Más información acerca del manejo de excepciones 404 y utilizar respuestas personalizadas para estos errores puede ser encontrada en la sección de [errores](/docs/errors#handling-404-errors) de la documentación.

<a name="routing-to-controllers"></a>
## Rutas con controladores

Laravel permite no solo definir rutas con Clausuras, sino también con clases de Controladores, incluso permite la creación de [controladores de recursos](/docs/controllers#resource-controllers).

Consulta la documentación en [Controladores](/docs/controllers) para más detalles.