# Vistas & Respuestas

- [Respuestas básicas](#basic-responses)
- [Redirecciones](#redirects)
- [Vistas](#views)
- [Composición de Vistas](#view-composers)
- [Respuestas especiales](#special-responses)
- [Macros de respuestas](#response-macros)

<a name="basic-responses"></a>
## Respuestas básicas

#### Retornando cadenas desde rutas

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### Creando respuestas personalizadas

Una instancia de `Response` herede la clase `Symfony\Component\HttpFoundation\Response`, proporcionando una variedad de métodos para crear respuestas HTTP.

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

Si necesitas acceder a los métodos de la clase `Response`, pero deseas retornar una vista como el contenido de la respuesta, puedes usar el método `Response::view` convenientemente:

	return Response::view('hello')->header('Content-Type', $type);

#### Adjuntando Cookies a respuestas

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## Redirecciones

#### Retornando una redirección

	return Redirect::to('user/login');

#### Retornando una redirección con datos instantáneos

	return Redirect::to('user/login')->with('message', 'Login Failed');

> **Nota:** Dado que el método `with` guarda datos instantáneos en la sesión, estos pueden ser obtenidos usando el método `Session:get`.

#### Retornando una redirección a una ruta con nombre

	return Redirect::route('login');

#### Retornando una redirección a una ruta con nombre con parámetros

	return Redirect::route('profile', array(1));

#### Retornando una redirección a una ruta con nombre con parámetros con nombres

	return Redirect::route('profile', array('user' => 1));

#### Retornando una redirección a la acción de un controlador

	return Redirect::action('HomeController@index');

#### Retornando una redireción a la acción de un controlador con parámetros

	return Redirect::action('UserController@profile', array(1));

#### Retornando una redirección a la acción de un controlador con parámetros con nombres

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## Vistas

Las vistas normalmente contienen el HTML de tu aplicación y proveen una manera conveniente para separar tu controlador y la lógica del dominio de tu lógica de presentación. Las archivos de las vistas son guardadas en el directorio `app/views`.

Una simple vista puede verse así:

	<!-- View stored in app/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

Esta vista puede ser retornada al navegador así:

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Taylor'));
	});

El segundo parámetro pasado al método `View::make` es un arreglo de datos que estarás disponibles en la vista:

#### Pasando datos a la vista

	// Using conventional approach
	$view = View::make('greeting')->with('name', 'Steve');

	// Using Magic Methods
	$view = View::make('greeting')->withName('steve');

En el ejemplo anterior la variable `$name` será accesible desde la vista, y será igual a `Steve`.

Si lo deseas, puedes pasar un arreglo de datos como segundo parámetro al método `make`

	$view = View::make('greetings', $data);

También puedes hacer disponible un segmento de datos en todas tus vistas:

	View::share('name', 'Steve');

#### Pasando una sub-vista a una vista

Algunas veces puedes desea pasar una vista dentro de otra vista. Por ejemplo, dada una sub-vista guardada en `app/views/child/view.php`, podemos pasarlal a otra vista así:

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

La sub-vista puede ser mostrada desde la vista padre:

	<html>
		<body>
			<h1>Hello!</h1>
			<?php echo $child; ?>
		</body>
	</html>

<a name="view-composers"></a>
## Composición de vistas

La composición de vistas son retrollamadas o métodos de clase que son llamados cuando una vista is mostrada. Si tienes datos que desees que se carguen cada vez que una vista es mostrada en tu aplicación, la composición de vistas pueden organizar ese código en un solo lugar. Por lo tanto, la composición de una vista pueden funcionar como 'Modelos de vistas' o 'presentadores'.

#### Definiendo el compositor de una vista

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

Ahora, cada vez que la vista `profile` se muestra, el dato `count` será cargado a la vista.

También puedes adjuntar un compositor de una vista a varias vistas en uno solo:

    View::composer(array('profile','dashboard'), function($view)
    {
        $view->with('count', User::count());
    });

Si prefieres usar una clase como compositor, lo cual provee el beneficio de ser resuelta a través de la aplicación gracias al [contenedor IoC](/docs/ioc), lo harías así:

	View::composer('profile', 'ProfileComposer');

Un compositos de una vista debería ser definido así:

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}

	}

#### Definiciendo múltiples compositores

Puedes usar el método `composers` para registar un grupo de compositores al mismo tiempo:

	View::composers(array(
		'AdminComposer' => array('admin.index', 'admin.profile'),
		'UserComposer' => 'user',
	));

> **Nota:** No hay una convención sobre el lugar donde las clases de compositores deberían estar. Existe total libertad para guardarlas donde desees mientras se pueden autocargar usando la configuración de tu archivo `composer.json`.

### Creadores de Vistas

**Creadores** de vistas funcionan casi igual a los compositores de vistas; sin embargo, ellos son ejecutados inmediatamente cuando la vista es instanciada. Para registrar una creados de vistas, simplemente usa el método `creator`:

	View::creator('profile', function($view)
	{
		$view->with('count', User::count());
	});

<a name="special-responses"></a>
## Respuestas especiales

#### Creando una respuesta JSON

	return Response::json(array('name' => 'Steve', 'state' => 'CA'));

#### Creando una respuesta JSONP

	return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));

#### Creando una respuesta para descarga de archivo

	return Response::download($pathToFile);

	return Response::download($pathToFile, $name, $headers);

> **Nota:** Symfony HttpFoundation, el cual maneja la descarga de archivos, requiere que el archivo sea descargado con un nombre ASCII.

<a name="response-macros"></a>
## Macros de respuestas

Si quieres definir una respuesta personalizada que puedes reutilizar en tus rutas y controladores, puedes usar el método `Response::macro`:

	Response::macro('caps', function($value)
	{
		return Response::make(strtoupper($value));
	});

La función `macro` acepta un nombre como primer parámetro y una Clausura como segundo parámetro. La Clausura del macro será ejecutado cuando se llame el nombre del macro desde la clase `Response`:

	return Response::caps('foo');

Puedes definir tus macros en uno de los archivos del directorio `app/start`. Alternativamente, puedes organizar tus macros en un archivo separado que sea incluído por algunos de los archivos del directorio `app/start.`