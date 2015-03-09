# Hashing

- [Introducción](#introduction)
- [Uso Básico](#basic-usage)

<a name="introduction"></a>
## Introducción

El Facade 'Hash' de Laravel provee el uso de Bcrypt hash seguro para almacenar contraseñas de usuarios. Si usted esta usando el controlador 'AuthController' que esta incluido en su aplicación de Laravel, el verificará la contraseña hasheada por Bcrypt contra la contraseña sin hashear suministrada por el usuario.

Del mismo modo, el servicio 'Registrar' que se incluye con Laravel hace la llamada 'bcrypt' a la funcion apropiada para verificar las contraseñas almacenadas.

<a name="basic-usage"></a>
## Uso Básico

#### Hasheando una contraseña usando Bcrypt

	$password = Hash::make('secret');

Tambien puedes usar la funcion helper 'bcrypt':

	$password = bcrypt('secret');

#### Verificando una contraseña contra una hasheada

	if (Hash::check('secret', $hashedPassword))
	{
		// Las contraseñas coincidieron
	}

#### Para verificar si una contraseña necesita ser rehasheada

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}
