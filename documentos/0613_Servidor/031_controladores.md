# 3.1. Controladores

Hasta el momento hemos visto solamente como devolver una cadena para una ruta y como asociar una vista a una ruta directamente en el fichero de rutas. Pero en general la forma recomendable de trabajar será **asociar dichas rutas a un método de un controlador**. Esto nos permitirá separar mucho mejor el código y crear clases (_controladores_) que agrupen toda la funcionalidad de un determinado recurso. Por ejemplo, _podemos crear un controlador para gestionar toda la lógica asociada al control de usuarios o cualquier otro tipo de recurso_.

Como ya vimos en la [sección de introducción](./01_introduccion.md), los _controladores_ son el punto de entrada de las peticiones de los usuarios y son los que **deben contener toda la lógica asociada al procesamiento de una petición**, encargándose de realizar las consultas necesarias a la base de datos, de preparar los datos y de llamar a la vista correspondiente con dichos datos.

## Controlador básico

Los controladores se almacenan en _ficheros PHP_ en la carpeta `app/Http/Controllers` y normalmente se les añade el sufijo _Controller_, por ejemplo `UserController.php` o `MoviesController.php`. A continuación se incluye un ejemplo básico de un controlador almacenado en el fichero `app/Http/Controllers/UserController.php`:

```
<?php
namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    /**
     * Mostrar información de un usuario.
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = User::findOrFail($id);
        return view('user.profile', ['user' => $user]);
    }
}
```

Todos los controladores tienen que extender la clase base `Controller`. Esta clase viene ya creada por defecto con la instalación de _Laravel_, la podemos encontrar en la carpeta `app/Http/Controllers`. Se utiliza para centralizar toda la lógica que se vaya a utilizar de forma compartida por los controladores de nuestra aplicación. Por defecto solo carga código para validación y autorización, pero podemos añadir en la misma todos los métodos que necesitemos.

En el código de ejemplo, el método `showProfile($id)` lo único que realiza es obtener los datos de un usuario, generar la vista `user.profile` a partir de los datos obtenidos y devolverla como valor de retorno para que se muestre por pantalla.

Una vez definido un _controlador_ ya podemos asociarlo a una _ruta_. Para esto tenemos que modificar el fichero de rutas `routes.php` de la forma:

```
use App\Http\Controllers\UserController;
Route::get('user/{id}', [UserController::class, 'showProfile']);
```

En lugar de pasar una función como segundo parámetro, tenemos que escribir una cadena que contenga el _nombre del controlador_, seguido de una arroba `@` y del _nombre del método_ que queremos asociar. No es necesario añadir nada más, ni los parámetros que recibe el método en cuestión, todo esto se hace de forma automática.

## Crear un nuevo controlador

Como hemos visto los controladores se almacenan dentro de la carpeta app/Http/Controllers como ficheros PHP. Para crear uno nuevo bien lo podemos hacer a mano y rellenar nosotros todo el código, o podemos utilizar el siguiente comando de Artisan que nos adelantará todo el trabajo:

```php artisan make:controller ProyectosController```

Este comando creará el controlador `ProyectosController` dentro de la carpeta `app/Http/Controllers` y lo completará con el código básico que hemos visto antes.

## Controladores y espacios de nombres

También podemos crear _sub-carpetas_ dentro de la carpeta `Controllers` para organizarnos mejor. En este caso, la estructura de carpetas que creemos no tendrá nada que ver con la ruta asociada a la petición y, de hecho, a la hora de hacer referencia al controlador únicamente tendremos que hacerlo a través de su espacio de nombres.

Como hemos visto al referenciar el _controlador_ en el fichero de rutas únicamente tenemos que indicar su nombre y no toda la ruta ni el espacio de nombres `App\Http\Controllers`. Esto es porque el servicio encargado de cargar las rutas añade automáticamente el espacio de nombres raíz para los controladores. Si metemos todos nuestros controladores dentro del mismo espacio de nombres no tendremos que añadir nada más. Pero si decidimos crear sub-carpetas y organizar nuestros controladores en sub-espacios de nombres, entonces sí que tendremos que añadir esa parte.

Por ejemplo, si creamos un controlador en `App\Http\Controllers\Photos\AdminController`, entonces para registrar una ruta hasta dicho controlador tendríamos que hacer:

```Route::get('foo', 'Photos\AdminController@method');```

## Generar una URL a una acción

Para generar la URL que apunte a una acción de un controlador podemos usar el método action de la forma:

```$url = action('FooController@method');```

Por ejemplo, para crear en una plantilla con Blade un enlace que apunte a una acción haríamos:

```<a href="{{ action('FooController@method') }}">¡Aprieta aquí!</a>```

## Caché de rutas

Si definimos todas nuestras rutas para que utilicen controladores podemos aprovechar la nueva funcionalidad para crear una _caché de las rutas_. Es importante que estén basadas en controladores porque si definimos respuestas directas desde el fichero de rutas (como vimos en el capítulo anterior) la caché no funcionará.

Gracias a la caché Laravel indican que se puede acelerar el proceso de registro de rutas hasta 100 veces. Para generar la caché simplemente tenemos que ejecutar el comando de Artisan:

```php artisan route:cache```

Si creamos más rutas y queremos añadirlas a la caché simplemente tenemos que volver a lanzar el mismo comando. Para borrar la caché de rutas y no generar una nueva caché tenemos que ejecutar:

```php artisan route:clear```

La caché se recomienda crearla solo cuando ya vayamos a pasar a producción nuestra web. Cuando estamos trabajando en la web es posible que añadamos nuevas rutas y sino nos acordamos de regenerar la caché la ruta no funcionará.
