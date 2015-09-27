# Autenticación

- [Introducción](#introduction)
- [Inicio rápido](#authentication-quickstart)
    - [Rutas](#included-routing)
    - [Vistas](#included-views)
    - [Autenticar](#included-authenticating)
    - [Obtener el usuario autenticado](#retrieving-the-authenticated-user)
    - [Proteger rutas](#protecting-routes)
    - [Regular autenticación](#authentication-throttling)
- [Autenticar usuarios manualmente](#authenticating-users)
    - [Recordar usuarios](#remembering-users)
    - [Otros métodos de autenticación](#other-authentication-methods)
- [Authenticación Básica HTTP](#http-basic-authentication)
     - [Authenticación Básica HTTP sin estado](#stateless-http-basic-authentication)
- [Restablecer contraseñas](#resetting-passwords)
    - [Consideraciones en la BD](#resetting-database)
    - [Rutas](#resetting-routing)
    - [Vistas](#resetting-views)
    - [Después de resetear contraseñas](#after-resetting-passwords)
- [Autenticación social](#social-authentication)
- [Agregar métodos de autenticación personalizados](#adding-custom-authentication-drivers)

<a name="introduction"></a>
## Introducción

Laravel hace que implementar la autenticación de tu aplicación sea muy fácil. De hecho, casi todo es configurado por ti desde el inicio. El archivo de configuración de la autenticación se encuentra en `config/auth.php`, el cual contiene varias opciones muy bien documentadas para personalizar el comportamiento de los servicios de autenticación.

### Consideración de la base de datos

De forma predeterminada, Laravel incluye el [modelo](//{{version}}/eloquent) de Eloquent `App\User` en la carpeta `app`. Este model puede ser usado con el driver predeterminado de Eloqeunt. Si tu aplicación no esta usando Eloquent, puedes usar el driver de autenticación `database` el cual usa el constructor de consultas de Laravel.

Mientras estes definiendo el esquema de base de datos para el modelo `App\User`, asegúrate que la columna de la contraseña tenga al menos 60 carácteres de longitud.

Además, debes verificar que tu tabla `users` (o equivalente) contenga una columna `remember_token` tipo string, permita valores nulos y una longitud de 100 carácteres. Ésta columna será usada para guardar el token para "recordar" las sesiones de tu aplicación. Esta columna puede ser creada en las migraciones usando `$table->rememberToken();`.

<a name="authentication-quickstart"></a>
## Guía de inicio rápido de autenticación

Laravel incluye 2 controladores de autenticación de forma predeterminada, los cuales están localizados en el namespace `App\Http\Controller\Auth`. El controlador `AuthController` se encarga de manejar el registro de nuevos usuarios y su autenticación, mientras el controlador `PasswordController` contiene la lógica para resetear la contraseña de usuarios existentes. Cada uno de esos controladores usan un `trait` para incluir sus métodos necesarios. Para muchas aplicaciones, no necesitarás modificar estos controladores.

<a name="included-routing"></a>
### Rutas

De forma predeterminada, no hay [rutas](/{{version}}/routing) para dirigir las peticiones a los controladores de autenticación. Puedes agregarlas manualmente a tu archivo `app/Http/routes.php`:

    // Rutas de autenticación...
    Route::get('auth/login', 'Auth\AuthController@getLogin');
    Route::post('auth/login', 'Auth\AuthController@postLogin');
    Route::get('auth/logout', 'Auth\AuthController@getLogout');

    // Rutas de registro...
    Route::get('auth/register', 'Auth\AuthController@getRegister');
    Route::post('auth/register', 'Auth\AuthController@postRegister');

<a name="included-views"></a>
### Vistas

Aunque los controladores de autenticación están incluídos en el framework, necesitas proveer las [vistas](/{{version}}/views) que esos controladores van a mostrar. Las vistas deben estar en la carpeta `resources/views/auth`. Eres libre de personalizarlas como necesites. La vista de inicio de sesión debe estar en `resources/views/auth/login.blade.php`, y la vista de registro debe estar en `resources/views/auth/register.blade.php`.

#### Ejemplo de formulario de autenticación

    <!-- resources/views/auth/login.blade.php -->

    <form method="POST" action="/auth/login">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password" id="password">
        </div>

        <div>
            <input type="checkbox" name="remember"> Remember Me
        </div>

        <div>
            <button type="submit">Login</button>
        </div>
    </form>

#### Ejemplo de formulario de registro

    <!-- resources/views/auth/register.blade.php -->

    <form method="POST" action="/auth/register">
        {!! csrf_field() !!}

        <div>
            Name
            <input type="text" name="name" value="{{ old('name') }}">
        </div>

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div>
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">Register</button>
        </div>
    </form>

<a name="included-authenticating"></a>
### Autenticar

Ahora que tienes tus rutas y vistas configuradas para los controladores de autenticación, estas listo para registrar y autenticar nuevos usuarios en tu aplicación. Puedes simplemente acceder a las rutas desde tu navegador. Los controladores de autenticación ya contienen la lógica (a través de los traits) para autenticar usuarios existentes y guardar los nuevos en la base de datos.

Cuando un usuario se autentica correctamente, serán redirigidos a la URI `/home`, la cual necesitas registrar como ruta. Puedes personalizar la redirección después de autenticar definiendo la propiedad `redirectPath` en el controlador `AuthController`:

    protected $redirectPath = '/dashboard';

Cuando un usuario no se autentica correctamente, serán redirigidos a la URI `/auth/login`. Puedes personalizar esta dirección definiendo la propiedad `loginPath` en el controlador `AuthController`:

    protected $loginPath = '/login';

La propiedad `loginPath` no cambiará cuando un usuario trata de acceder una ruta protegida. Eso es controlado por el método `handle` en el middleware `App\Http\Middleware\Authenticate`.

#### Personalización

Para modificar los campos del formulario que son requeridos cuando un nuevo usuario se registra en tu aplicación, o personalizar como se guardan los registro de nuevos usuarios en la base de datos, necesitas modificar el controlador `AuthController`. Esta clase es responsable de validar y crear nuevos usuarios en tu aplicación.

El método `validator` del controlador `AuthController` contiene las reglas de validación de nuevos usuarios para tu aplicación. Eres libre de modificar éste método como creas conveniente.

El método `create` del controlador `AuthController` es responsable de crear nuevos registros `App\User` en tu base de datos usando el [ORM Eloquent](/{{version}}/eloquent). Eres libre de modificar éste método como creas mas conveniente.

<a name="retrieving-the-authenticated-user"></a>
### Obtener el usuario autenticado

Puedes acceder al usuario autenticado a través del facade `Auth`:

    $user = Auth::user();

Alternativamente, una vez el usuario se encuentre autenticado, puedes acceder al usuario usando una instancia de `Illuminate\Http\Request`:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() returns an instance of the authenticated user...
            }
        }
    }

#### Determinar si el usuario actual esta autenticado

Para determinar si el usuario ya se encuentra autenticado en tu aplicación, puedes usar el método `check` del facade `Auth`, el cual retornará `true` si el usuario está autenticado:

    if (Auth::check()) {
        // The user is logged in...
    }

Sin embargo, puedes usar un middleware para verificar que el usuario esta autenticado antes de permitirle acceder a ciertas rutas o controladores. Para aprender más sobre esto, mira la documentación sobre [protección de rutas](/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Proteger rutas

[Rutas middleware](/{{version}}/middleware) pueden ser usadas para permitir solamente usuarios autenticados acceder a ciertas rutas. Laravel incluye el middleware `auth`, el cual esta definido en `app\Http\Middleware\Authenticate.php`. Todo lo que necesitas es agregar el middleware a la definición de la ruta:

    // Using A Route Closure...

    Route::get('profile', ['middleware' => 'auth', function() {
        // Only authenticated users may enter...
    }]);

    // Using A Controller...

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

Por supuesto, si estas usando [controladores](/{{version}}/controllers), puedes llamar al método `middleware` desde el constructor del controlador en vez de agregar a la definiciòn de la ruta:

    public function __construct()
    {
        $this->middleware('auth');
    }

<a name="authentication-throttling"></a>
### Regular autenticación

Si estas usando la clase `AuthController` que viene con Laravel, el trait `Illuminate\Foundatio\Auth\ThrottlesLogins` puede ser usado para regular los intentos de inicio de sesión en tu aplicación. De forma predeterminada, el usuario no podrá iniciar sesión por un minuto si falla en ingresar los datos correctos varias veces. La regulación es única para cada combinación usuario/email y si dirección IP.

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // Rest of AuthController class...
    }

<a name="authenticating-users"></a>
## Autenticando usuarios manualmente

Por supuesto, no estas obligado a usar los controladores de autenticación de Laravel. Si decides eliminar esos controladores, necesitarás manejar la autenticación de usuarios usando las clases de autenticación de Laravel directamente. No te preocupes, es muy sencillo:

Vamos a hacer uso de los servicios de autenticación de Laravel a través del [facade](/{{version}}/facades) `Auth`, asegúrate de importar el facade `Auth` al inicio de la clase. A continuación, revisemos el método `attempt`:

    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

El método `attempt` accepta un array de pares llaves/valores como primer argumento. Los valores en el array serán usados para encontrar el usuario en la tabla de tu base de datos. En el ejemplo anterior, el usuario será consultado por el valor de la columna `email`. Si el usuario es encontrado, la contraseña cifrada guardada en la base de datos será comparada con el valor cifrado de `password` en el array. Si las dos contraseñas cifradas concuerdan una sesión de autenticación se creará para el usuario.

El método `attempt` retornará `true` si la autenticación fue satisfactoria. Si no, retornará `false`.

El método `intended` en el redireccionador redireccionará el usuario a la URL que estaban tratando de acceder antes de ser filtrados por la autenticación. Una URI puede ser usada como respaldo en este método en el caso que el destino final no este disponible.

Si lo deseas, puedes agregar condiciones extras a la consulta de autenticación de usuario y contraseña. Por ejemplo, puedes verificar que el usuario quede como "activo" en la aplicación:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

Para cerrar la sesión de un usuario en tu aplicación, puedes usar el método `logout` del facade `Auth`. Esto limpiará la información de autenticación del usuario de la sesión:

    Auth::logout();

> **Nota:** En estos ejemplos, `email` no es una opción requerida, es usada como ejemplo. Debes usar cualquier columna que corresponda con el "usuario" de tu aplicación.

<a name="remembering-users"></a>
## Recordar usuarios

Si quieres proveer la funcionalidad "Recordarme" en tu aplicación, puedes pasar un valor booleano como el segundo parametro del método `attempt`, el cual mantendrá el usuario autenticado indefinidamente, o hasta que cierre sesión manualmente. Por supuesto, tu tabla `users` debe incluir una columna tipo `string` llamada `remember_token`, la cual será usada para guardar el token "recordarme".

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

Si eras "recordando" usuarios, puedes usar el método `viaRemember` para determinar si el usuario se autenticó con esa opción:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Otros métodos de autenticación

#### Autenticar una instancia de usuario

Si necesitas autenticar una instancia existente de un usuario en tu aplicación, puedes ejecutar el método `login` con la instancia del usuario. El objeto dado debe ser una implementación del [contrato](/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Por supuesto, el modelo `App\User` incluído por Laravel ya implementa esta interface:

    Auth::login($user);

#### Autenticar un usuario por ID

Para autenticar un usuario en tu aplicación por su ID, puedes usar el método `loginUsingId`. Este método simplemente acepta la llave primaria del usuario que deseas autenticar:

    Auth::loginUsingId(1);

#### Autenticar un usuario una sola vez

Puedes usar el método `once` para autenticar el usuario durante una sola petición. No se usarán sesiones o cookies, algo muy útil cuando estas construyendo una API sin estado. El método `once` recibe los mismos argumentos que `attempt`:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## Autenticación básica HTTP

[Autenticación básica HTTP](http://en.wikipedia.org/wiki/Basic_access_authentication) provee una forma rápida para autenticar usuarios en tu aplicación sin configurar una página de inicio de sesión. Para iniciar, incluye el [middleware](/{{version}}/middleware) `auth.basic` a tu ruta. El middleware `auth.basic` viene incluido con Laravel, asi que no necesitas definirlo:

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // Only authenticated users may enter...
    }]);

Una vez el middleware este incluido en la ruta, automaticamente se te solicitará las credenciales de acceso cuando trates de acceder a la ruta. De forma predeterminada, el middleware `auth.basic` usará la columna `email` del registro de usuario como "nombre de usuario".

#### Nota sobre FastCGI

Si estás usando PHP FastCGI, la autenticación básica HTTP puede no funcionar correctamente. Las siguientes lineas deberián ser agregadas a tu archivo `.htaccess`:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Autenticación básica HTTP sin estado

Puedes usar la Autenticación básica HTTP sin configurar una cookie para identificar el usuario en la sesión, algo particularmente útil para autenticación de APIs. Para hacerlo, debes definir un [middleware](/{{version}}/middleware) que llame al método `onceBasic`. Si ninguna respuesta es retornado por el método `onceBasic`, la petición se aceptará en la aplicación:

    <?php

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

A continuación, se debe [registrar el middleware de la ruta](/{{version}}/middleware#registering-middleware) y agregarla a la ruta:

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // Only authenticated users may enter...
    }]);

<a name="resetting-passwords"></a>
## Restablecer contraseñas

<a name="resetting-database"></a>
### Consideración de la base de datos

La mayoría de aplicaciones web proveen una forma para que sus usuarios restablezcan sus contraseñas. En vez de hacerte reimplementar esta funcionalidad en cada aplicación, Laravel provee método convenientes para enviar recordatorios de contraseñas y restablecerlas.

Para iniciar, verifica que tu modelo `App\User` implementa el contrato `Illuminate\Contracts\Auth\CanResetPassword`. Por supuesto, el modelo `App\User` incluído con el framework ya tiene implementado éste contrato, y usa el trait `Illuminate\Auth\Passwords\CanResetPassword` para incluir los métodos necesarios para implementar la contrato.

#### Generar la tabla para los tokens de reseteo

A continuación, una tabla debe ser creada para almanecar los tokens de restablecimiento de contraseña. La migración para esta tabla esta incluída de forma predeterminada en Laravel, y se encuentra en la carpeta `database/migrations`. Así, todo lo que necesitas es:

    php artisan migrate

<a name="resetting-routing"></a>
### Rutas

Laravel incluye el controlador `Auth\PasswordController` que contiene la lógica necesaria para restablecer la contraseña de usuarios. Sin embargo, necesitas definir las rutas a que apunten a este controlador:

    // Password reset link request routes...
    Route::get('password/email', 'Auth\PasswordController@getEmail');
    Route::post('password/email', 'Auth\PasswordController@postEmail');

    // Password reset routes...
    Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
    Route::post('password/reset', 'Auth\PasswordController@postReset');

<a name="resetting-views"></a>
### Vistas

Además de definir las rutas para `PasswordController`, necesitas establecer las vistas que serán usadas por este controlador. No te preocupues, nosotros proveemos vistas de ejemplo para ayudarte a empezar. Por supuesto, eres libre de estilisarlas como quieras.

#### Ejemplo de formulario para solicitar reseteo de contraseña

Necesitas una vista HTML para el formulario de solicitud de contraseña. Esta vista debe estar en `resources/views/auth/password.blade.php`. Este formulario provee un solo campo para el email del usuario, permitiendole solicitar un enlace de restablecimiento de contraseña:

    <!-- resources/views/auth/password.blade.php -->

    <form method="POST" action="/password/email">
        {!! csrf_field() !!}

        @if (count($errors) > 0)
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        @endif

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <button type="submit">
                Send Password Reset Link
            </button>
        </div>
    </form>

Cuando un usuario envia una petición para restablecer su contraseña, recibirá un email con un enlace que apunta al método `getReset` (normalmente en la ruta `/password/reset`) del controlador `PasswordController`. Necesitas crear una vista para este email en `resouces/views/emails/password.blade.php`. La vista recibirá la variable `$token` que contiene el token de restablecimiento de contraseña para ser comparado con la petición del usuario. Acá hay un ejemplo la vista de email para empezar:

    <!-- resources/views/emails/password.blade.php -->

    Click here to reset your password: {{ url('password/reset/'.$token) }}

#### Ejemplo formulario reseteo de contraseña

Cuando el usuario hace click en el enlace enviado a su email para restablecer su contraseña, se le mostrará un formulario de restablecimiento de contraseña. Esta vista debe estar en `resources/views/auth/reset.blade.php`.

Acá un ejemplo del formulario de restablecimiento de contraseña para empezar:

    <!-- resources/views/auth/reset.blade.php -->

    <form method="POST" action="/password/reset">
        {!! csrf_field() !!}
        <input type="hidden" name="token" value="{{ $token }}">
        @if (count($errors) > 0)
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        @endif

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div>
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">
                Reset Password
            </button>
        </div>
    </form>

<a name="after-resetting-passwords"></a>
### Después de restablecer contraseña

Una vez hayas definido las rutas y las vistas para restablecer la contraseña de los usuarios, puedes acceder a la ruta en tu navegador. El controlador `PasswordController` incluido con el framework ya tiene la lógica para enviar los emails con los enlaces de restablecimineto de contraseña asi como la actualización de las contraseñas en la base de datos.

Después de que la contraseña es restablecida, el usuario automáticamente sería autenticado en la aplicación y redirigido a la ruta `/home`. Puedes personalizar la ruta de redirección definiendo la propiedad `redirectTo` en el controlador `PasswordController`:

    protected $redirectTo = '/dashboard';

> **Note:** By default, password reset tokens expire after one hour. You may change this via the `reminder.expire` option in your `config/auth.php` file.

<a name="social-authentication"></a>
## Autenticación Social

In addition to typical, form based authentication, Laravel also provides a simple, convenient way to authenticate with OAuth providers using [Laravel Socialite](https://github.com/laravel/socialite). Socialite currently supports authentication with Facebook, Twitter, LinkedIn, Google, GitHub and Bitbucket.

To get started with Socialite, add to your `composer.json` file as a dependency:

    composer require laravel/socialite

### Configuración

After installing the Socialite library, register the `Laravel\Socialite\SocialiteServiceProvider` in your `config/app.php` configuration file:

    'providers' => [
        // Other service providers...

        Laravel\Socialite\SocialiteServiceProvider::class,
    ],

Also, add the `Socialite` facade to the `aliases` array in your `app` configuration file:

    'Socialite' => Laravel\Socialite\Facades\Socialite::class,

You will also need to add credentials for the OAuth services your application utilizes. These credentials should be placed in your `config/services.php` configuration file, and should use the key `facebook`, `twitter`, `linkedin`, `google`, `github` or `bitbucket`, depending on the providers your application requires. For example:

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

### Uso básico

Next, you are ready to authenticate users! You will need two routes: one for redirecting the user to the OAuth provider, and another for receiving the callback from the provider after authentication. We will access Socialite using the `Socialite` [facade](/{{version}}/facades):

    <?php

    namespace App\Http\Controllers;

    use Socialite;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

The `redirect` method takes care of sending the user to the OAuth provider, while the `user` method will read the incoming request and retrieve the user's information from the provider. Before redirecting the user, you may also set "scopes" on the request using the `scope` method. This method will overwrite all existing scopes:

    return Socialite::driver('github')
                ->scopes(['scope1', 'scope2'])->redirect();

Of course, you will need to define routes to your controller methods:

        Route::get('auth/github', 'Auth\AuthController@redirectToProvider');
        Route::get('auth/github/callback', 'Auth\AuthController@handleProviderCallback');

A number of OAuth providers support optional parameters in the redirect request. To include any optional parameters in the request, call the `with` method with an associative array:

    return Socialite::driver('google')
                ->with(['hd' => 'example.com'])->redirect();

#### Obtener detalles de usuario

Once you have a user instance, you can grab a few more details about the user:

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="adding-custom-authentication-drivers"></a>
## Agregar drivers de autenticación personalizados

If you are not using a traditional relational database to store your users, you will need to extend Laravel with your own authentication driver. We will use the `extend` method on the `Auth` facade to define a custom driver. You should place this call to `extend` within a [service provider](/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('riak', function($app) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...
                return new RiakUserProvider($app['riak.connection']);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

After you have registered the driver with the `extend` method, you may switch to the new driver in your `config/auth.php` configuration file.

### The User Provider Contract

The `Illuminate\Contracts\Auth\UserProvider` implementations are only responsible for fetching a `Illuminate\Contracts\Auth\Authenticatable` implementation out of a persistent storage system, such as MySQL, Riak, etc. These two interfaces allow the Laravel authentication mechanisms to continue functioning regardless of how the user data is stored or what type of class is used to represent it.

Let's take a look at the `Illuminate\Contracts\Auth\UserProvider` contract:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

The `retrieveById` function typically receives a key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.

The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. The new token can be either a fresh token, assigned on successful "remember me" login attempt, or a null when user is logged out.

The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `UserInterface`. **This method should not attempt to do any password validation or authentication.**

The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method might compare the `$user->getAuthPassword()` string to a `Hash::make` of `$credentials['password']`. This method should only validate the user's credentials and return boolean.

### The Authenticatable Contract

Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable` contract. Remember, the provider should return implementations of this interface from the `retrieveById` and `retrieveByCredentials` methods:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

This interface is simple. The `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.