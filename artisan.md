# Artisan CLI

- [Introducción](#introduction)
- [Uso](#usage)

<a name="introduction"></a>
## Introducción

Artisan es el nombre de la línea de comandos incluída en Laravel. Esta proporciona una serie de comandos útiles para tu uso mientras desarrollas tu aplicación. Ésta es implusada por el potente componente 'Symfony Console'.

<a name="usage"></a>
## Uso

#### Listar Todos los Comandos Disponibles

Para ver la lista de los comandos disponibles en Artisan puedes usar el comando `list`:

	php artisan list

#### Ver la Pantalla de Ayuda para un Comando

Cada comando además incluye una pantalla de "ayuda" que muestra y describe los argumentos y opciones disponibles de un comando. Para ver la ayuda, simplemente agrega `help` antes del nombre del comando:

	php artisan help migrate

#### Especificar la Configuración del Entorno

Puedes espeficar el entorno de configuración que debería usarse para correr el comando usando la opción `--env`:

	php artisan migrate --env=local

#### Mostrar Tu Versión Actual de Laravel

Puedes ver la versión actual de tu instalación de Laravel usando la opción `--version`:

	php artisan --version
