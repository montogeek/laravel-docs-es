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
- [Resetear contraseñas](#resetting-passwords)
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

Of course, you are not required to use the authentication controllers included with Laravel. If you choose to remove these controllers, you will need to manage user authentication using the Laravel authentication classes directly. Don't worry, it's a cinch!

We will access Laravel's authentication services via the `Auth` [facade](/{{version}}/facades), so we'll need to make sure to import the `Auth` facade at the top of the class. Next, let's check out the `attempt` method:

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

The `attempt` method accepts an array of key / value pairs as its first argument. The values in the array will be used to find the user in your database table. So, in the example above, the user will be retrieved by the value of the `email` column. If the user is found, the hashed password stored in the database will be compared with the hashed `password` value passed to the method via the array. If the two hashed passwords match an authenticated session will be started for the user.

The `attempt` method will return `true` if authentication was successful. Otherwise, `false` will be returned.

The `intended` method on the redirector will redirect the user to the URL they were attempting to access before being caught by the authentication filter. A fallback URI may be given to this method in case the intended destination is not available.

If you wish, you also may add extra conditions to the authentication query in addition to the user's e-mail and password. For example, we may verify that user is marked as "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

To log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

    Auth::logout();

> **Note:** In these examples, `email` is not a required option, it is merely used as an example. You should use whatever column name corresponds to a "username" in your database.

<a name="remembering-users"></a>
## Recordar usuarios

If you would like to provide "remember me" functionality in your application, you may pass a boolean value as the second argument to the `attempt` method, which will keep the user authenticated indefinitely, or until they manually logout. Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

If you are "remembering" users, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Otros métodos de autenticación

#### Autenticar una instancia de usuario

If you need to log an existing user instance into your application, you may call the `login` method with the user instance. The given object must be an implementation of the `Illuminate\Contracts\Auth\Authenticatable` [contract](/{{version}}/contracts). Of course, the `App\User` model included with Laravel already implements this interface:

    Auth::login($user);

#### Autenticar un usuario por ID

To log a user into the application by their ID, you may use the `loginUsingId` method. This method simply accepts the primary key of the user you wish to authenticate:

    Auth::loginUsingId(1);

#### Autenticar un usuario una sola vez

You may use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized, which may be helpful when building a stateless API. The `once` method has the same signature as the `attempt` method:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## Autenticación básica HTTP

[HTTP Basic Authentication](http://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` [middleware](/{{version}}/middleware) to your route. The `auth.basic` middleware is included with the Laravel framework, so you do not need to define it:

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // Only authenticated users may enter...
    }]);

Once the middleware has been attached to the route, you will automatically be prompted for credentials when accessing the route in your browser. By default, the `auth.basic` middleware will use the `email` column on the user record as the "username".

#### Nota sobre FastCGI

If you are using PHP FastCGI, HTTP Basic authentication may not work correctly out of the box. The following lines should be added to your `.htaccess` file:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Autenticación básica HTTP sin estado

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, [define a middleware](/{{version}}/middleware) that calls the `onceBasic` method. If no response is returned by the `onceBasic` method, the request may be passed further into the application:

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

Next, [register the route middleware](/{{version}}/middleware#registering-middleware) and attach it to a route:

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // Only authenticated users may enter...
    }]);

<a name="resetting-passwords"></a>
## Resetear contraseñas

<a name="resetting-database"></a>
### Consideración de la base de datos

Most web applications provide a way for users to reset their forgotten passwords. Rather than forcing you to re-implement this on each application, Laravel provides convenient methods for sending password reminders and performing password resets.

To get started, verify that your `App\User` model implements the `Illuminate\Contracts\Auth\CanResetPassword` contract. Of course, the `App\User` model included with the framework already implements this interface, and uses the `Illuminate\Auth\Passwords\CanResetPassword` trait to include the methods needed to implement the interface.

#### Generar la tabla para los tokens de reseteo

Next, a table must be created to store the password reset tokens. The migration for this table is included with Laravel out of the box, and resides in the `database/migrations` directory. So, all you need to do is migrate:

    php artisan migrate

<a name="resetting-routing"></a>
### Rutas

Laravel includes an `Auth\PasswordController` that contains the logic necessary to reset user passwords. However, you will need to define routes to point requests to this controller:

    // Password reset link request routes...
    Route::get('password/email', 'Auth\PasswordController@getEmail');
    Route::post('password/email', 'Auth\PasswordController@postEmail');

    // Password reset routes...
    Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
    Route::post('password/reset', 'Auth\PasswordController@postReset');

<a name="resetting-views"></a>
### Vistas

In addition to defining the routes for the `PasswordController`, you will need to provide views that can be returned by this controller. Don't worry, we will provide sample views to help you get started. Of course, you are free to style your forms however you wish.

#### Ejemplo de formulario para solicitar reseteo de contraseña

You will need to provide an HTML view for the password reset request form. This view should be placed at `resources/views/auth/password.blade.php`. This form provides a single field for the user's e-mail address, allowing them to request a password reset link:

    <!-- resources/views/auth/password.blade.php -->

    <form method="POST" action="/password/email">
        {!! csrf_field() !!}

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

When a user submits a request to reset their password, they will receive an e-mail with a link that points to the `getReset` method (typically routed at `/password/reset`) of the `PasswordController`. You will need to create a view for this e-mail at `resources/views/emails/password.blade.php`. The view will receive the `$token` variable which contains the password reset token to match the user to the password reset request. Here is an example e-mail view to get you started:

    <!-- resources/views/emails/password.blade.php -->

    Click here to reset your password: {{ url('password/reset/'.$token) }}

#### Ejemplo formulario reseteo de contraseña

When the user clicks the e-mailed link to reset their password, they will be presented with a password reset form. This view should be placed at `resources/views/auth/reset.blade.php`.

Here is a sample password reset form to get you started:

    <!-- resources/views/auth/reset.blade.php -->

    <form method="POST" action="/password/reset">
        {!! csrf_field() !!}
        <input type="hidden" name="token" value="{{ $token }}">

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
### Después de resetear contraseña

Once you have defined the routes and views to reset your user's passwords, you may simply access the routes in your browser. The `PasswordController` included with the framework already includes the logic to send the password reset link e-mails as well as update passwords in the database.

After the password is reset, the user will automatically be logged into the application and redirected to `/home`. You can customize the post password reset redirect location by defining a `redirectTo` property on the `PasswordController`:

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