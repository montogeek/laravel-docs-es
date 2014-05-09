# Peticiones y Entrada de datos

- [Entrada de datos básico](#basic-input)
- [Cookies](#cookies)
- [Entrada de datos antigua](#old-input)
- [Archivos](#files)
- [Solicitud de información](#request-information)

<a name="basic-input"></a>
## Entrada de datos básico

Puedes acceder a los datos ingresados por el usuario con varios métodos muy simples. No tienes necesidad de preocuparte por el verbo HTTP usado en la petición, la entrada de datos se accede de la misma manera para todos los verbos.

#### Accediendo al valor de una entrada de datos

	$name = Input::get('name');

#### Accediendo a un valor predeterminado si la entrada de datos no existe

	$name = Input::get('name', 'Sally');

#### Comprobar si una entrada de datos existe

	if (Input::has('name'))
	{
		//
	}

#### Obtener toda la entrada de datos de la petición

	$input = Input::all();

#### Obtener alguna entrada de datos de la petición

	$input = Input::only('username', 'password');

	$input = Input::except('credit_card');

Cuando tengas formularios con entrada de datos como arreglos, puedes usar la notación de punto para acceder los arreglos:

	$input = Input::get('products.0.name');

> **Nota:** Algunas librerías JavaScript como Backbone envían la entrada de datos en formato JSON. Puedes acceder a ellos naturalmente con `Input::get`.

<a name="cookies"></a>
## Cookies

Todas las cookies creadas por Laravel son escriptadas y firmadas con un código de autentificación, esto significa que serán consideradas inválidas si fueran cambiadas por el ciente.

#### Accediendo al valor de una cookie

	$value = Cookie::get('name');

#### Adjuntar una cookie nueva a una respuesta

	$response = Response::make('Hello World');

	$response->withCookie(Cookie::make('name', 'value', $minutes));

#### Encolar una cookie para la siguiente respuesta

Si deseas establecer una cookie antes de que una respuesta se haya creado, usa el método `Cookie::queue()`. La cookie automáticamente se adjuntará a la respuesta final de tu aplicación:

	Cookie::queue($name, $value, $minutes);

#### Creando una cookie que dure para siempre

	$cookie = Cookie::forever('name', 'value');

<a name="old-input"></a>
## Entrada de datos antigua

Puedes necesitar mantener la entrada de datos de una petición hasta la siguiente petición. Por ejemplo, cuando necesites repoblar un formulario después de comprobar si tiene errores de validación.

#### Guardando entrada de datos en la sesión

	Input::flash();

#### Guardando aguna entrada de datos a la sesión

	Input::flashOnly('username', 'email');

	Input::flashExcept('password');

Ya que a menudo desearás guardar la entrada de datos combinada con una redirección a la página anterior, puedes hacerlo fácilmente encadenando la guardada de la entrada de datos en una redirección:

	return Redirect::to('form')->withInput();

	return Redirect::to('form')->withInput(Input::except('password'));

> **Nota:** Puedes guardar otros datos entre peticiones usando la clase [Sesión](/docs/session).

#### Obteniendo datos antiguos

	Input::old('username');

<a name="files"></a>
## Archivos

#### Obteniendo un archivo subido

	$file = Input::file('photo');

#### Determinar si un archivo fue subido

	if (Input::hasFile('photo'))
	{
		//
	}

El objeto retornado por el métodot `file` es una instancia de la clase `Symfony\Component\HttpFoundation\File\UploadedFile`, la cual extiende la clase PHP `SplFileInfo` y provee una variedad de métodos para interactuar con el archivo.

#### Moviendo un archivo subido

	Input::file('photo')->move($destinationPath);

	Input::file('photo')->move($destinationPath, $fileName);

#### Obteniendo la ruta de un archivo subido

	$path = Input::file('photo')->getRealPath();

#### Obteniendo el nombre original de un archivo subido

	$name = Input::file('photo')->getClientOriginalName();

#### Obteniendo la extensión de un archivo subido

	$extension = Input::file('photo')->getClientOriginalExtension();

#### Obteniendo el tamaño de un archivo subido

	$size = Input::file('photo')->getSize();

#### Obteniendo el tipo MIME de un archivo subido

	$mime = Input::file('photo')->getMimeType();

<a name="request-information"></a>
## Solicitud de información

La clase `Request` provee diversos métodos para examinar la petición HTTP de tu aplicación y extiende la clase `Symfony\Component\HttpFoundation\Request`.
Algunos de los métodos más relevantes:

#### Obteniendo la URI de la petición

	$uri = Request::path();

#### Obteniendo el método de la petición

	$method = Request::method();

	if (Request::isMethod('post'))
	{
		//
	}

#### Determinar si la ruta de la petición concuerda con un patrón

	if (Request::is('admin/*'))
	{
		//
	}

#### Obtener la URL de la petición

	$url = Request::url();

#### Obtener un segmento de la URI de la petición

	$segment = Request::segment(1);

#### Obtener un encabezado de la petición

	$value = Request::header('Content-Type');

#### Obtener valores de $_SERVER

	$value = Request::server('PATH_INFO');

#### Determinar si la petición es HTTPS

	if (Request::secure())
	{
		//
	}

#### Determinar si la petición es AJAX

	if (Request::ajax())
	{
		//
	}

#### Determinar si la petición tiene tipo de contenido JSON

	if (Request::isJson())
	{
		//
	}

#### Determinar si la petición espera un contenido tipo JSON

	if (Request::wantsJson())
	{
		//
	}

#### Verificar el formato de respuesta de la petición

El método `Request::format` retornará el formato de respuesta de la petición basado en el encabezado HTTP Accept

	if (Request::format() == 'json')
	{
		//
	}
