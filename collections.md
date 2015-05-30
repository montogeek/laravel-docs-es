# Colecciones

- [Introduccion](#introduction)
- [Uso basico](#basic-usage)

<a name="introduction"></a>
## Introduccion

La clase `Illuminate\Support\Collection` provee una forma conviniene y fluida para trabajar con arrays de datos. Por ejemplo, revisa el siguiente codigo. Se usa el metodo `collect` para crear una nueva instancia de una coleccion a partir de un array:

  $collection = collect(['taylor', 'abigail', null])->map(function($name)
  {
    return strtoupper($name);
  })
  ->reject(function($name)
  {
    return empty($value);
  });


Como puedes ver, la clase `Collection` te permite encadenar sus metodos para ejecutar mapeos o reducciones sobre el array. En general, cada metodo de `Collection` retorna una completa nueva instancia de `Colecction`. Aprende mas a continuacion!

<a name="basic-usage"></a>
## Uso basico

#### Crear colecciones

Como se menciono anteriormente, el metodo `collect` retornara una nueva instancia de `Illuminate\Support\Collection` dado un array. Tambien puedes usar el metodo `make` de la clase `Collection`

  $collection = collect([1, 2, 3]);

  $collection = Collection::make([1, 2, 3]);

Por supuesto, colecciones de los objetos de [Eloquent](/5.0/eloquent) siempre son retornados como instancias de `Collection`, sin embargo, usa la clase `Collection` donde sea que la necesites en tu aplicacion.

#### Explorar la coleccion

En vez de listar todos los metodos (son muchos) que la clase Collection provee, verifica la [documentacion en la API](http://laravel.com/api/master/Illuminate/Support/Collection.html)!