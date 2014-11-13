# Artisan CLI

- [Introducción](#introduction)
- [Uso](#usage)

<a name="introduction"></a>
## Introducción

Artisan es el nombre de la utilidad para la línea de comandos incluída en Laravel. Provee varios comandos útiles para ser usados mientras desarrollas tu aplicación. It is driven by the powerful Symfony Console component.

<a name="usage"></a>
## Uso

Para ver una lista de los comandos disponibles en Artisan puedes usar el comando `list`:

#### Mostrar lista de todos los comandos disponibles

	php artisan list

Cada comando además incluye una pantalla de ayuda que muestra y describe los argumentos y opciones disponibles del comando. Para ver la ayuda, simplemente agrega `help` antes del nombre del comando:

#### Ver la ayuda de un comando

	php artisan help migrate

Puedes espeficar el entorno de configuración que debería usarse para correr el comando usando la opción `--env`:

#### Especificar la configuración del entorno

	php artisan migrate --env=local

Puedes ver la versión de Laravel usando la opción `--version`:

#### Mostrar la versión de Laravel

	php artisan --version
