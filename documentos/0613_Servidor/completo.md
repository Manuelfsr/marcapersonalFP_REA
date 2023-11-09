4. Base de datos

Laravel facilita la configuración y el uso de diferentes tipos de base de datos: MySQL, Postgres, SQLite y SQL Server. En el fichero de configuración (config/database.php) tenemos que indicar todos los parámetros de acceso a nuestras bases de datos y además especificar cual es la conexión que se utilizará por defecto. En Laravel podemos hacer uso de varias bases de datos a la vez, aunque sean de distinto tipo. Por defecto se accederá a la que especifiquemos en la configuración y si queremos acceder a otra conexión lo tendremos que indicar expresamente al realizar la consulta.

En este capítulo veremos como configurar una base de datos, como crear tablas y especificar sus campos desde código, como inicializar la base de datos y como construir consultas tanto de forma directa como a través del ORM llamado Eloquent.
4.1. Configuración inicial

En este primer apartado vamos a ver los primeros pasos que tenemos que dar con Laravel para empezar a trabajar con bases de datos. Para esto vamos a ver a continuación como definir la configuración de acceso, como crear una base de datos y como crear la tabla de migraciones, necesaria para crear el resto de tablas.
Configuración de la Base de Datos

Lo primero que tenemos que hacer para trabajar con bases de datos es completar la configuración. Como ejemplo vamos a configurar el acceso a una base de datos tipo MySQL. Si editamos el fichero con la configuración config/database.php podemos ver en primer lugar la siguiente línea:

'default' => env('DB_CONNECTION', 'mysql'),

Este valor indica el tipo de base de datos a utilizar por defecto. Como vimos en el primer capítulo Laravel utiliza el sistema de variables de entorno para separar las distintas configuraciones de usuario o de máquina. El método env('DB_CONNECTION', 'mysql') lo que hace es obtener el valor de la variable DB_CONNECTION del fichero .env. En caso de que dicha variable no esté definida devolverá el valor por defecto mysql.

En este mismo fichero de configuración, dentro de la sección connections, podemos encontrar todos los campos utilizados para configurar cada tipo de base de datos, en concreto la base de datos tipo mysql tiene los siguientes valores:

'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'), // Nombre de la base de datos
    'username'  => env('DB_USERNAME', 'forge'), // Usuario de acceso a la bd
    'password'  => env('DB_PASSWORD', ''),      // Contraseña de acceso
    'charset'   => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'prefix'    => '',
    'strict'    => false,
],

Como se puede ver, básicamente los campos que tenemos que configurar para usar nuestra base de datos son: host, database, username y password. El host lo podemos dejar como está si vamos a usar una base de datos local, mientras que los otros tres campos sí que tenemos que actualizarlos con el nombres de la base de datos a utilizar y el usuario y la contraseña de acceso. Para poner estos valores abrimos el fichero .env de la raíz del proyecto y los actualizamos:

DB_CONNECTION=mysql
DB_HOST=localhost
DB_DATABASE=nombre-base-de-datos
DB_USERNAME=nombre-de-usuario
DB_PASSWORD=contraseña-de-acceso

Crear la base de datos

Para crear la base de datos que vamos a utilizar en MySQL podemos utilizar la herramienta PHPMyAdmin que se ha instalado con el paquete XAMPP. Para esto accedemos a la ruta:

http://localhost/phpmyadmin

La cual nos mostrará un panel para la gestión de las bases de datos de MySQL, que nos permite, además de realizar cualquier tipo de consulta SQL, crear nuevas bases de datos o tablas, e insertar, modificar o eliminar los datos directamente. En nuestro caso apretamos en la pestaña "Bases de datos" y creamos una nueva base de datos. El nombre que le pongamos tiene que ser el mismo que el que hayamos indicado en el fichero de configuración de Laravel.
Tabla de migraciones

A continuación vamos a crear la tabla de migraciones. En la siguiente sección veremos en detalle que es esto, de momento solo decir que Laravel utiliza las migraciones para poder definir y crear las tablas de la base de datos desde código, y de esta manera tener un control de las versiones de las mismas.

Para poder empezar a trabajar con las migraciones es necesario en primer lugar crear la tabla de migraciones. Para esto tenemos que ejecutar el siguiente comando de Artisan:

php artisan migrate:install

    Si nos diese algún error tendremos que revisar la configuración que hemos puesto de la base de datos y si hemos creado la base de datos con el nombre, usuario y contraseña indicado.

Si todo funciona correctamente ahora podemos ir al navegador y acceder de nuevo a nuestra base de datos con PHPMyAdmin, podremos ver que se nos habrá creado la tabla migrations. Con esto ya tenemos configurada la base de datos y el acceso a la misma. En las siguientes secciones veremos como añadir tablas y posteriormente como realizar consultas.
4.2. Migraciones

Las migraciones son un sistema de control de versiones para bases de datos. Permiten que un equipo trabaje sobre una base de datos añadiendo y modificando campos, manteniendo un histórico de los cambios realizados y del estado actual de la base de datos. Las migraciones se utilizan de forma conjunta con la herramienta Schema builder (que veremos en la siguiente sección) para gestionar el esquema de base de datos de la aplicación.

La forma de funcionar de las migraciones es crear ficheros (PHP) con la descripción de la tabla a crear y posteriormente, si se quiere modificar dicha tabla se añadiría una nueva migración (un nuevo fichero PHP) con los campos a modificar. Artisan incluye comandos para crear migraciones, para ejecutar las migraciones o para hacer rollback de las mismas (volver atrás).
Crear una nueva migración

Para crear una nueva migración se utiliza el comando de Artisan make:migration, al cual le pasaremos el nombre del fichero a crear y el nombre de la tabla:

php artisan make:migration create_users_table --create=users

Esto nos creará un fichero de migración en la carpeta database/migrations con el nombre <TIMESTAMP>_create_users_table.php. Al añadir un timestamp a las migraciones el sistema sabe el orden en el que tiene que ejecutar (o deshacer) las mismas.

Si lo que queremos es añadir una migración que modifique los campos de una tabla existente tendremos que ejecutar el siguiente comando:

php artisan make:migration add_votes_to_user_table --table=users

En este caso se creará también un fichero en la misma carpeta, con el nombre <TIMESTAMP>_add_votes_to_user_table.php pero preparado para modificar los campos de dicha tabla.

Por defecto, al indicar el nombre del fichero de migraciones se suele seguir siempre el mismo patrón (aunque el realidad el nombre es libre). Si es una migración que crea una tabla el nombre tendrá que ser create_<table-name>_table y si es una migración que modifica una tabla será <action>_to_<table-name>_table.
Estructura de una migración

El fichero o clase PHP generada para una migración siempre tiene una estructura similar a la siguiente:

<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration 
{
    /**
     * Run the migrations.
     * @return void
     */
    public function up()
    {
        //
    }

    /**
     * Reverse the migrations.
     * @return void
     */
    public function down()
    {
        //
    }
}

En el método up es donde tendremos crear o modificar la tabla, y en el método down tendremos que deshacer los cambios que se hagan en el up (eliminar la tabla o eliminar el campo que se haya añadido). Esto nos permitirá poder ir añadiendo y eliminando cambios sobre la base de datos y tener un control o histórico de los mismos.
Ejecutar migraciones

Después de crear una migración y de definir los campos de la tabla (en la siguiente sección veremos como especificar esto) tenemos que lanzar la migración con el siguiente comando:

php artisan migrate

    Si nos aparece el error "class not found" lo podremos solucionar llamando a composer dump-autoload y volviendo a lanzar las migraciones.

Este comando aplicará la migración sobre la base de datos. Si hubiera más de una migración pendiente se ejecutarán todas. Para cada migración se llamará a su método up para que cree o modifique la base de datos. Posteriormente en caso de que queramos deshacer los últimos cambios podremos ejecutar:

php artisan migrate:rollback

# O si queremos deshacer todas las migraciones
php artisan migrate:reset

Un comando interesante cuando estamos desarrollando un nuevo sitio web es migrate:refresh, el cual deshará todos los cambios y volver a aplicar las migraciones:

php artisan migrate:refresh

Además si queremos comprobar el estado de las migraciones, para ver las que ya están instaladas y las que quedan pendientes, podemos ejecutar:

php artisan migrate:status

4.3. Schema Builder

Una vez creada una migración tenemos que completar sus métodos up y down para indicar la tabla que queremos crear o el campo que queremos modificar. En el método down siempre tendremos que añadir la operación inversa, eliminar la tabla que se ha creado en el método up o eliminar la columna que se ha añadido. Esto nos permitirá deshacer migraciones dejando la base de datos en el mismo estado en el que se encontraban antes de que se añadieran.

Para especificar la tabla a crear o modificar, así como las columnas y tipos de datos de las mismas, se utiliza la clase Schema. Esta clase tiene una serie de métodos que nos permitirá especificar la estructura de las tablas independientemente del sistema de base de datos que utilicemos.
Crear y borrar una tabla

Para añadir una nueva tabla a la base de datos se utiliza el siguiente constructor:

Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});

Donde el primer argumento es el nombre de la tabla y el segundo es una función que recibe como parámetro un objeto del tipo Blueprint que utilizaremos para configurar las columnas de la tabla.

En la sección down de la migración tendremos que eliminar la tabla que hemos creado, para esto usaremos alguno de los siguientes métodos:

Schema::drop('users');

Schema::dropIfExists('users');

Al crear una migración con el comando de Artisan make:migration ya nos viene este código añadido por defecto, la creación y eliminación de la tabla que se ha indicado y además se añaden un par de columnas por defecto (id y timestamps).
Añadir columnas

El constructor Schema::create recibe como segundo parámetro una función que nos permite especificar las columnas que va a tener dicha tabla. En esta función podemos ir añadiendo todos los campos que queramos, indicando para cada uno de ellos su tipo y nombre, y además si queremos también podremos indicar una serie de modificadores como valor por defecto, índices, etc. Por ejemplo:

Schema::create('users', function($table)
{
    $table->increments('id');
    $table->string('username', 32);
    $table->string('password');
    $table->smallInteger('votos');
    $table->string('direccion');
    $table->boolean('confirmado')->default(false);
    $table->timestamps();
});

Schema define muchos tipos de datos que podemos utilizar para definir las columnas de una tabla, algunos de los principales son:
Comando	Tipo de campo
$table->boolean('confirmed'); 	BOOLEAN
$table->enum('choices', array('foo', 'bar')); 	ENUM
$table->float('amount'); 	FLOAT
$table->increments('id'); 	Clave principal tipo INTEGER con Auto-Increment
$table->integer('votes'); 	INTEGER
$table->mediumInteger('numbers'); 	MEDIUMINT
$table->smallInteger('votes'); 	SMALLINT
$table->tinyInteger('numbers'); 	TINYINT
$table->string('email'); 	VARCHAR
$table->string('name', 100); 	VARCHAR con la longitud indicada
$table->text('description'); 	TEXT
$table->timestamp('added_on'); 	TIMESTAMP
$table->timestamps(); 	Añade los timestamps "created_at" y "updated_at"
->nullable() 	Indicar que la columna permite valores NULL
->default($value) 	Declare a default value for a column
->unsigned() 	Añade UNSIGNED a las columnas tipo INTEGER

Los tres últimos se pueden combinar con el resto de tipos para crear, por ejemplo, una columna que permita nulos, con un valor por defecto y de tipo unsigned.

Para consultar todos los tipos de datos que podemos utilizar podéis consultar la documentación de Laravel en:

http://laravel.com/docs/migrations#columns
Añadir índices

Schema soporta los siguientes tipos de índices:
Comando	Descripción
$table->primary('id'); 	Añadir una clave primaria
$table->primary(array('first', 'last')); 	Definir una clave primaria compuesta
$table->unique('email'); 	Definir el campo como UNIQUE
$table->index('state'); 	Añadir un índice a una columna

En la tabla se especifica como añadir estos índices después de crear el campo, pero también permite indicar estos índices a la vez que se crea el campo:

$table->string('email')->unique();

Claves ajenas

Con Schema también podemos definir claves ajenas entre tablas:

$table->bigInteger('user_id')->unsigned();
$table->foreign('user_id')->references('id')->on('users');

En este ejemplo en primer lugar añadimos la columna "user_id" de tipo UNSIGNED INTEGER (siempre tendremos que crear primero la columna sobre la que se va a aplicar la clave ajena). A continuación creamos la clave ajena entre la columna "user_id" y la columna "id" de la tabla "users".

    La columna con la clave ajena tiene que ser del mismo tipo que la columna a la que apunta. Si por ejemplo creamos una columna a un índice auto-incremental tendremos que especificar que la columna sea unsigned para que no se produzcan errores.

También podemos especificar las acciones que se tienen que realizar para "on delete" y "on update":

$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');

Para eliminar una clave ajena, en el método down de la migración tenemos que utilizar el siguiente código:

$table->dropForeign('posts_user_id_foreign');

Para indicar la clave ajena a eliminar tenemos que seguir el siguiente patrón para especificar el nombre <tabla>_<columna>_foreign. Donde "tabla" es el nombre de la tabla actual y "columna" el nombre de la columna sobre la que se creo la clave ajena.
4.4. Modelos de datos mediante ORM

El mapeado objeto-relacional (más conocido por su nombre en inglés, Object-Relational mapping, o por sus siglas ORM) es una técnica de programación para convertir datos entre un lenguaje de programación orientado a objetos y una base de datos relacional como motor de persistencia. Esto posibilita el uso de las características propias de la orientación a objetos, podremos acceder directamente a los campos de un objeto para leer los datos de una base de datos o para insertarlos o modificarlos.

Laravel incluye su propio sistema de ORM llamado Eloquent, el cual nos proporciona una manera elegante y fácil de interactuar con la base de datos. Para cada tabla de la base datos tendremos que definir su correspondiente modelo, el cual se utilizará para interactuar desde código con la tabla.
Definición de un modelo

Por defecto los modelos se guardarán como clases PHP dentro de la carpeta app/Models, sin embargo Laravel nos da libertad para colocarlos en otra carpeta si queremos, como por ejemplo la carpeta app/Models. Pero en este caso tendremos que asegurarnos de indicar correctamente el espacio de nombres.

Para definir un modelo que use Eloquent únicamente tenemos que crear una clase que herede de la clase Model:

<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    //...
}

Sin embargo es mucho más fácil y rápido crear los modelos usando el comando make:model de Artisan:

php artisan make:model User

Este comando creará el fichero User.php dentro de la carpeta app con el código básico de un modelo que hemos visto en el ejemplo anterior.
Convenios en Eloquent
Nombre

En general el nombre de los modelos se pone en singular con la primera letra en mayúscula, mientras que el nombre de las tablas suele estar en plural. Gracias a esto, al definir un modelo no es necesario indicar el nombre de la tabla asociada, sino que Eloquent automáticamente buscará la tabla transformando el nombre del modelo a minúsculas y buscando su plural (en inglés). En el ejemplo anterior que hemos creado el modelo User buscará la tabla de la base de datos llamada users y en caso de no encontrarla daría un error.

Si la tabla tuviese otro nombre lo podemos indicar usando la propiedad protegida $table del modelo:

<?php 
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'my_users';
}

Clave primaria

Laravel también asume que cada tabla tiene declarada una clave primaria con el nombre id. En el caso de que no sea así y queramos cambiarlo tendremos que sobrescribir el valor de la propiedad protegida $primaryKey del modelo, por ejemplo: protected $primaryKey = 'my_id';.

    Es importante definir correctamente este valor ya que se utiliza en determinados métodos de Eloquent, como por ejemplo para buscar registros o para crear las relaciones entre modelos.

Timestamps

Otra propiedad que en ocasiones tendremos que establecer son los timestamps automáticos. Por defecto Eloquent asume que todas las tablas contienen los campos updated_at y created_at (los cuales los podemos añadir muy fácilmente con Schema añadiendo $table->timestamps() en la migración). Estos campos se actualizarán automáticamente cuando se cree un nuevo registro o se modifique. En el caso de que no queramos utilizarlos (y que no estén añadidos a la tabla) tendremos que indicarlo en el modelo o de otra forma nos daría un error. Para indicar que no los actualice automáticamente tendremos que modificar el valor de la propiedad pública $timestamps a false, por ejemplo: public $timestamps = false;.

A continuación se muestra un ejemplo de un modelo de Eloquent en el que se añaden todas las especificaciones que hemos visto:

<?php 
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected $table = 'my_users';
    protected $primaryKey = 'my_id'
    public $timestamps = false;
}

Uso de un modelo de datos

Una vez creado el modelo ya podemos empezar a utilizarlo para recuperar datos de la base de datos, para insertar nuevos datos o para actualizarlos. El sitio correcto donde realizar estas acciones es en el controlador, el cual se los tendrá que pasar a la vista ya preparados para su visualización.

Es importante que para su utilización indiquemos al inicio de la clase el espacio de nombres del modelo o modelos a utilizar. Por ejemplo, si vamos a usar los modelos User y Orders tendríamos que añadir:

use App\Models\User;
use App\Models\Orders;

Consultar datos

Para obtener todas las filas de la tabla asociada a un modelo usaremos el método all():

$users = User::all();

foreach( $users as $user ) {
    echo $user->name;
}

Este método nos devolverá un array de resultados, donde cada item del array será una instancia del modelo User. Gracias a esto al obtener un elemento del array podemos acceder a los campos o columnas de la tabla como si fueran propiedades del objeto ($user->name).

    Nota: Todos los métodos que se describen en la sección de "Constructor de consultas" y en la documentación de Laravel sobre "Query Builder" también se pueden utilizar en los modelos Eloquent. Por lo tanto podremos utilizar where, orWhere, first, get, orderBy, groupBy, having, skip, take, etc. para elaborar las consultas.

Eloquent también incorpora el método find($id) para buscar un elemento a partir del identificador único del modelo, por ejemplo:

$user = User::find(1);
echo $user->name;

Si queremos que se lance una excepción cuando no se encuentre un modelo podemos utilizar los métodos findOrFail o firstOrFail. Esto nos permite capturar las excepciones y mostrar un error 404 cuando sucedan.

$model = User::findOrFail(1);

$model = User::where('votes', '>', 100)->firstOrFail();

A continuación se incluyen otros ejemplos de consultas usando Eloquent con algunos de los métodos que ya habíamos visto en la sección "Constructor de consultas":

// Obtener 10 usuarios con más de 100 votos
$users = User::where('votes', '>', 100)->take(10)->get();

// Obtener el primer usuario con más de 100 votos
$user = User::where('votes', '>', 100)->first();

También podemos utilizar los métodos agregados para calcular el total de registros obtenidos, o el máximo, mínimo, media o suma de una determinada columna. Por ejemplo:

$count = User::where('votes', '>', 100)->count();
$price = Orders::max('price');
$price = Orders::min('price');
$price = Orders::avg('price');
$total = User::sum('votes');

Insertar datos

Para añadir una entrada en la tabla de la base de datos asociada con un modelo simplemente tenemos que crear una nueva instancia de dicho modelo, asignar los valores que queramos y por último guardarlos con el método save():

$user = new User;
$user->name = 'Juan';
$user->save();

Para obtener el identificador asignado en la base de datos después de guardar (cuando se trate de tablas con índice auto-incremental), lo podremos recuperar simplemente accediendo al campo id del objeto que habíamos creado, por ejemplo:

$insertedId = $user->id;

Actualizar datos

Para actualizar una instancia de un modelo es muy sencillo, solo tendremos que recuperar en primer lugar la instancia que queremos actualizar, a continuación modificarla y por último guardar los datos:

$user = User::find(1);
$user->email = 'juan@gmail.com';
$user->save();

Borrar datos

Para borrar una instancia de un modelo en la base de datos simplemente tenemos que usar su método delete():

$user = User::find(1);
$user->delete();

Si por ejemplo queremos borrar un conjunto de resultados también podemos usar el método delete() de la forma:

$affectedRows = User::where('votes', '>', 100)->delete();

Más información

Para más información sobre como crear relaciones entre modelos, eager loading, etc. podéis consultar directamente la documentación de Laravel en:

http://laravel.com/docs/eloquent
4.5. Inicialización de la base de datos (Database Seeding)

Laravel también facilita la inserción de datos iniciales o datos semilla en la base de datos. Esta opción es muy útil para tener datos de prueba cuando estamos desarrollando una web o para crear tablas que ya tienen que contener una serie de datos en producción.

Los ficheros de "semillas" se encuentran en la carpeta database/seeders. Por defecto Laravel incluye el fichero DatabaseSeeder con el siguiente contenido:

<?php

use Illuminate\Database\Seeder;
use Illuminate\Database\Eloquent\Model;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     * @return void
     */
    public function run()
    {
        //...
    }
}

Al lanzar la incialización se llamará por defecto al método run de la clase DatabaseSeeder. Desde aquí podemos crear las semillas de varias formas:

    Escribir el código para insertar los datos dentro del propio método run.
    Crear otros métodos dentro de la clase DatabaseSeeder y llamarlos desde el método run. De esta forma podemos separar mejor las inicializaciones.
    Crear otros ficheros Seeder y llamarlos desde el método run es la clase principal.

Según lo que vayamos a hacer nos puede interesar una opción u otra. Por ejemplo, si el código que vamos a escribir es poco nos puede sobrar con las opciones 1 o 2, sin embargo si vamos a trabajar bastante con las inicializaciones quizás lo mejor es la opción 3.

A continuación se incluye un ejemplo de la opción 1:

<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeders.
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
        ]);
    }
}

Como se puede ver en el ejemplo en general tendremos que eliminar primero los datos de la tabla en cuestión y posteriormente añadir los datos. Para insertar datos en una tabla podemos utilizar el "Constructor de consultas" y "Eloquent ORM".
Crear ficheros semilla

Como hemos visto en el apartado anterior, podemos crear más ficheros o clases semilla para modularizar mejor el código de las inicializaciones. De esta forma podemos crear un fichero de semillas para cada una de las tablas o modelos de datos que tengamos.

En la carpeta database/seeders podemos añadir más ficheros PHP con clases que extiendan de Seeder para definir nuestros propios ficheros de "semillas". El nombre de los ficheros suele seguir el mismo patrón <nombre-tabla>TableSeeder, por ejemplo "UsersTableSeeder". Artisan incluye un comando que nos facilitará crear los ficheros de semillas y que además incluirán las estructura base de la clase. Por ejemplo, para crear el fichero de inicialización de la tabla de usuarios haríamos:

php artisan make:seeder UsersTableSeeder

Para que esta nueva clase se ejecute tenemos que llamarla desde el método run de la clase principal DatabaseSeeder de la forma:

class DatabaseSeeder extends Seeder 
{
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Support\Facades\Schema;

    public function run()
    {
        Model::unguard();
        Schema::disableForeignKeyConstraints();

        $this->call(UsersTableSeeder::class);

        Model::reguard();

        Schema::enableForeignKeyConstraints();
 }
}

El método call lo que hace es llamar al método run de la clase indicada. Además en el ejemplo hemos añadido las llamadas a unguard y a reguard, que lo que hacen es desactivar y volver a activar (respectivamente) la inserción de datos masiva o por lotes.
Ejecutar la inicialización de datos

Una vez definidos los ficheros de semillas, cuando queramos ejecutarlos para rellenar de datos la base de datos tendremos que usar el siguiente comando de Artisan:

php artisan db:seed

4.6. Constructor de consultas (Query Builder)

Laravel incluye una serie de clases que nos facilita la construcción de consultas y otro tipo de operaciones con la base de datos. Además, al utilizar estas clases, creamos una notación mucho más legible, compatible con todos los tipos de bases de datos soportados por Laravel y que nos previene de cometer errores o de ataques por inyección de código SQL.
Consultas

Para realizar una "Select" que devuelva todas las filas de una tabla utilizaremos el siguiente código:

$users = DB::table('users')->get();

foreach ($users as $user)
{
    echo $user->name;
}

En el ejemplo se utiliza el constructor DB::tabla indicando el nombre de la tabla sobre la que se va a realizar la consulta, y por último se llama al método get() para obtener todas las filas de la misma.

Si queremos obtener un solo elemento podemos utilizar first en lugar de get, de la forma:

$user = DB::table('users')->first();

echo $user->name;

Clausula where

Para filtrar los datos usamos la clausula where, indicando el nombre de la columna y el valor a filtrar:

$user = DB::table('users')->where('name', 'Pedro')->get();

echo $user->name;

En este ejemplo, la clausula where filtrará todas las filas cuya columna name sea igual a Pedro. Si queremos realizar otro tipo de filtrados, como columnas que tengan un valor mayor (>), mayor o igual (>=), menor (<), menor o igual (<=), distinto del indicado (<>) o usar el operador like, lo podemos indicar como segundo parámetro de la forma:

$users = DB::table('users')->where('votes', '>', 100)->get();

$users = DB::table('users')->where('status', '<>', 'active')->get();

$users = DB::table('users')->where('name', 'like', 'T%')->get();

Si añadimos más clausulas where a la consulta por defecto se unirán mediante el operador lógico AND. En caso de que queramos utilizar el operador lógico OR lo tendremos que realizar usando orWhere de la forma:

$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'Pedro')
                    ->get();

orderBy / groupBy / having_

También podemos utilizar los métodos orderBy, groupBy y having en las consultas:

$users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->groupBy('count')
                    ->having('count', '>', 100)
                    ->get();

Offset / Limit

Si queremos indicar un offset o limit lo realizaremos mediante los métodos skip (para el offset) y take (para limit), por ejemplo:

$users = DB::table('users')->skip(10)->take(5)->get();

Transacciones

Laravel también permite crear transacciones sobre un conjunto de operaciones:

DB::transaction(function()
{
    DB::table('users')->update(array('votes' => 1));

    DB::table('posts')->delete();
});

En caso de que se produzca cualquier excepción en las operaciones que se realizan en la transacción se desharían todos los cambios aplicados hasta ese momento de forma automática.
Más informacion

Para más información sobre la construcción de Querys (join, insert, update, delete, agregados, etc.) podéis consultar la documentación de Laravel en su sitio web:

http://laravel.com/docs/queries
4.7. Ejercicios

En estos ejercicios vamos a continuar con el proyecto del videoclub que habíamos empezado en sesiones anteriores y le añadiremos todo lo referente a la gestión de la base de datos.

Ejercicio 1 - Configuración de la base de datos y migraciones (1 punto)

En primer lugar vamos a configurar correctamente la base de datos. Para esto tenemos que actualizar el fichero .env para indicar que vamos a usar una base de datos tipo MySQL llamada "videoclub" junto con el nombre de usuario y contraseña de acceso.

A continuación abrimos PHPMyAdmin y creamos la nueva base de datos llamada videoclub. Para comprobar que todo se ha configurado correctamente vamos a un terminal en la carpeta de nuestro proyecto y ejecutamos el comando que crea la tabla de migraciones. Si todo va bien podremos actualizar desde PHPMyAdmin y comprobar que se ha creado esta tabla dentro de nuestra nueva base de datos.

    Si nos diese algún error tendremos que revisar los valores indicados en el fichero .env. En caso de ser correctos es posible que también tengamos que reiniciar el servidor o terminal que tengamos abierto.

Ahora vamos a crear la tabla que utilizaremos para almacenar el catálogo de películas. Ejecuta el comando de Artisan para crear la migración llamada create_movies_table para la tabla movies. Una vez creado edita este fichero para añadir todos los campos necesarios, estos son:

Campo	Tipo	Valor por defecto
id 	Autoincremental 	
title 	String 	
year 	Year 	
director 	String de longitud 64 	
poster 	String 	
rented 	Booleano 	false
synopsis 	Text 	
timestamps 	Timestamps de Eloquent 	 

    Recuerda que en el método down de la migración tienes que deshacer los cambios que has hecho en el método up, en este caso sería eliminar la tabla.

Por último ejecutaremos el comando de Artisan que añade las nuevas migraciones y comprobaremos que la tabla se ha creado correctamente con los campos que le hemos indicado.

Ejercicio 2 - Modelo de datos (0.5 puntos)

En este ejercicio vamos a crear el modelo de datos asociado con la tabla movies. Para esto usaremos el comando apropiado de Artisan para crear el modelo llamado Movie.

Una vez creado este fichero lo abriremos y comprobaremos que el nombre de la clase sea el correcto y que herede de la clase Model. Y ya está, no es necesario hacer nada más, el cuerpo de la clase puede estar vacío ({}), todo lo demás se hace automáticamente!

Ejercicio 3 - Semillas (1 punto)

Ahora vamos a proceder a rellenar la tabla de la base de datos con los datos iniciales. Para esto editamos el fichero de semillas situado en database/seeders/DatabaseSeeder.php y seguiremos los siguientes pasos:

    Creamos un método privado de clase llamado seedCatalog() que se tendrá que llamar desde el método run de la forma:

    public function run()
    {
      self::seedCatalog();
      $this->command->info('Tabla catálogo inicializada con datos!');
    }

    Movemos el array de películas que se facilitaba en los materiales y que habíamos copiado dentro del controlador CatalogController a la clase de semillas (DatabaseSeeder.php), guardándolo como variable privada de la clase.

    Dentro del nuevo método seedCatalog() realizamos las siguientes acciones:
        En primer lugar borramos el contenido de la tabla movies con Movie::truncate();.
        Y a continuación añadimos el siguiente código:

        foreach( self::$arrayPeliculas as $pelicula ) {
            $p = new Movie;
            $p->title = $pelicula['title'];
            $p->year = $pelicula['year'];
            $p->director = $pelicula['director'];
            $p->poster = $pelicula['poster'];
            $p->rented = $pelicula['rented'];
            $p->synopsis = $pelicula['synopsis'];
            $p->save();
        }

Por último tendremos que ejecutar el comando de Artisan que procesa las semillas y una vez realizado comprobaremos que se rellenado la tabla movies con el listado de películas.

    Si te aparece el error "Fatal error: Class 'Movie' not found" revisa si has indicado el espacio de nombres del modelo que vas a utilizar (use App\Models\Movie;).

Ejercicio 4 - Uso de la base de datos (1 punto)

En este último ejercicio vamos a actualizar los métodos del controlador CatalogController para que obtengan los datos desde la base de datos. Seguiremos los siguientes pasos:

    Modificar el método getIndex para que obtenga toda la lista de películas desde la base de datos usando el modelo Movie y que se pase a la vista ese listado.

    Modificar el método getShow para que obtenga la película pasada por parámetro usando el método findOrFail y se pase a la vista dicha película.

    Modificar el método getEdit para que obtenga la película pasada por parámetro usando el método findOrFail y se pase a la vista dicha película.

    Si al probarlo te aparece el error "Class 'App\Http\Controllers\Movie' not found" revisa si has indicado el espacio de nombres del modelo que vas a utilizar (use App\Models\Movie;).

Ya no necesitaremos más el array de películas ($arrayPeliculas) que habíamos puesto en el controlador, así que lo podemos comentar o eliminar.

Ahora tendremos que actualizar las vistas para que en lugar de acceder a los datos del array los obtenga del objeto con la película. Para esto cambiaremos en todos los sitios donde hayamos puesto $pelicula['campo'] por $pelicula->campo.

Además, en la vista catalog/index.blade.php, en vez de utilizar el índice del array ($key) como identificador para crear el enlace a catalog/show/{id}, tendremos que utilizar el campo id de la película ($pelicula->id). Lo mismo en la vista catalog/show.blade.php, para generar el enlace de editar película tendremos que añadir el identificador de la película a la ruta catalog/edit.
5. Datos de entrada y Control de usuarios

En este cuarto capítulo vamos a aprender como recoger los datos de entrada de formularios o de algún otro tipo de petición (como por ejemplo una petición de una API). También veremos como leer ficheros de entrada.

En la sección de control de usuarios se tratará todo lo referente a la gestión de los usuarios de una aplicación web, desde como crear la tabla de usuarios, como registrarlos, autenticarlos en la aplicación, cerrar la sesión o como proteger las partes privadas de nuestra aplicación de accesos no permitidos.
5.1. Datos de entrada
Laravel facilita el acceso a los datos de entrada del usuario a través de solo unos pocos métodos. No importa el tipo de petición que se haya realizado (POST, GET, PUT, DELETE), si los datos son de un formulario o si se han añadido a la query string, en todos los casos se obtendrán de la misma forma.

Para conseguir acceso a estos métodos Laravel utiliza inyección de dependencias. Esto es simplemente añadir la clase Request al constructor o método del controlador en el que lo necesitemos. Laravel se encargará de inyectar dicha dependencia ya inicializada y directamente podremos usar este parámetro para obtener los datos de entrada. A continuación se incluye un ejemplo:

<?php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $name = $request->input('nombre');

        //...
    }
}

En este ejemplo como se puede ver se ha añadido la clase Request como parámetro al método store. Laravel automáticamente se encarga de inyectar estas dependencias por lo que directamente podemos usar la variable $request para obtener los datos de entrada.

Si el método del controlador tuviera más parámetros simplemente los tendremos que añadir a continuación de las dependencias, por ejemplo:

public function edit(Request $request, $id)
{
    //...
}

A continuación veremos los métodos y datos que podemos obtener a partir de la variable $request.
Obtener los valores de entrada

Para obtener el valor de una variable de entrada usamos el método input indicando el nombre de la variable:

$name = $request->input('nombre');

// O simplemente....
$name = $request->nombre;

También podemos especificar un valor por defecto como segundo parámetro:

$name = $request->input('nombre', 'Pedro');

Comprobar si una variable existe

Si lo necesitamos podemos comprobar si un determinado valor existe en los datos de entrada:

if ($request->has('nombre'))
{
    //...
}

Obtener datos agrupados

O también podemos obtener todos los datos de entrada a la vez (en un array) o solo algunos de ellos:

// Obtener todos: 
$input = $request->all();

// Obtener solo los campos indicados: 
$input = $request->only('username', 'password');

// Obtener todos excepto los indicados: 
$input = $request->except('credit_card');

Obtener datos de un array

Si la entrada proviene de un input tipo array de un formulario (por ejemplo una lista de checkbox), si queremos podremos utilizar la siguiente notación con puntos para acceder a los elementos del array de entrada:

$input = $request->input('products.0.name');

JSON

Si la entrada está codificada formato JSON (por ejemplo cuando nos comunicamos a través de una API es bastante común) también podremos acceder a los diferentes campos de los datos de entrada de forma normal (con los métodos que hemos visto, por ejemplo: $nombre = $request->input('nombre');).
Ficheros de entrada

Laravel facilita una serie de clases para trabajar con los ficheros de entrada. Por ejemplo para obtener un fichero que se ha enviado en el campo con nombre photo y guardarlo en una variable, tenemos que hacer:

$file = $request->file('photo');

// O simplemente...
$file = $request->photo;

Si queremos podemos comprobar si un determinado campo tiene un fichero asignado:

if ($request->hasFile('photo')) {
    //...
}

El objeto que recuperamos con $request->file() es una instancia de la clase Symfony\Component\HttpFoundation\File\UploadedFile, la cual extiende la clase de PHP SplFileInfo (http://php.net/manual/es/class.splfileinfo.php), por lo tanto, tendremos muchos métodos que podemos utilizar para obtener datos del fichero o para gestionarlo.

Por ejemplo, para comprobar si el fichero que se ha subido es válido:

if ($request->file('photo')->isValid()) {
    //...
}

En la última versión de Laravel se ha incorporado una nueva librería que nos permite gestionar el acceso y escritura de ficheros en un almacenamiento. Lo interesante de esto es que nos permite manejar de la misma forma el almacenamiento en local, en Amazon S3 y en Rackspace Cloud Storage, simplemente lo tenemos que configurar en config/filesystems.php y posteriormente los podremos usar de la misma forma. Por ejemplo, para almacenar un fichero subido mediante un formulario tenemos que usar el método store indicando como parámetro la ruta donde queremos almacenar el fichero (sin el nombre del fichero):

$path = $request->photo->store('images');
$path = $request->photo->store('images', 's3');  // Especificar un almacenamiento

Estos métodos devolverán el path hasta el fichero almacenado de forma relativa a la raíz de disco configurada. Para el nombre del fichero se generará automáticamente un UUID (identificador único universal). Si queremos especificar nosotros el nombre tendríamos que usar el método storeAs:

$path = $request->photo->storeAs('images', 'filename.jpg');
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');

Otros métodos que podemos utilizar para recuperar información del fichero son:

// Obtener la ruta:
$path = $request->file('photo')->getRealPath();

// Obtener el nombre original:
$name = $request->file('photo')->getClientOriginalName();

// Obtener la extensión: 
$extension = $request->file('photo')->getClientOriginalExtension();

// Obtener el tamaño: 
$size = $request->file('photo')->getSize();

// Obtener el MIME Type:
$mime = $request->file('photo')->getMimeType();

5.2. Control de usuarios

Laravel incluye una serie de métodos y clases que harán que la implementación del control de usuarios sea muy rápida y sencilla. De hecho, casi todo el trabajo ya está hecho, solo tendremos que indicar donde queremos utilizarlo y algunos pequeños detalles de configuración.

Por defecto, al crear un nuevo proyecto de Laravel, ya se incluye todo lo necesario:

    La configuración predeterminada en config/auth.php.
    La migración para la base de datos de la tabla de usuarios con todos los campos necesarios.
    El modelo de datos de usuario (User.php) dentro de la carpeta app con toda la implementación necesaria.
    Los controladores para gestionar todas las acciones relacionadas con el control de usuarios (dentro de App\Http\Controllers\Auth).

Además de esto tendremos que ejecutar los siguientes comandos para generar las rutas y vistas necesarias para realizar el login, registro y para recuperar la contraseña con Laravel/Breeze.

composer update
composer require laravel/breeze:1.9.4 --dev

php artisan breeze:install

npm install
npm run dev
php artisan migrate

En los siguientes apartados vamos a ver en detalle cada uno de estos puntos, desde la configuración hasta los módulos, rutas y vistas por los que está compuesto. En las últimas secciones revisaremos también cómo utilizar este sistema para proteger nuestro sitio web.
Configuración inicial

La configuración del sistema de autenticación se puede encontrar en el fichero config/auth.php, el cual contiene varias opciones (bien documentadas) que nos permitirán, por ejemplo: cambiar el sistema de autenticación (que por defecto es a través de Eloquent), cambiar el modelo de datos usado para los usuarios (por defecto será User) y cambiar la tabla de usuarios (que por defecto será users). Si vamos a utilizar estos valores no será necesario que realicemos ningún cambio.

La migración de la tabla de usuarios (llamada users) también está incluida (ver carpeta database/migrations). Por defecto incluye todos los campos necesarios (ver el código siguiente), pero si necesitamos alguno más lo podemos añadir para guardar por ejemplo la dirección o el teléfono del usuario. A continuación se incluye el código de la función up de la migración:

Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->rememberToken();
    $table->timestamps();
});

Como se puede ver el nombre de la tabla es users, con un índice id autoincremental, y los campos de name, email, password, donde el campo email se establece como único para que no se puedan almacenar emails repetidos. Además se añaden los timestamps que usa Eloquent para almacenar automáticamente la fecha de registro y actualización, y el campo remember_token para recordar la sesión del usuario.

En la carpeta app se encuentra el modelo de datos (llamado User.php) para trabajar con los usuarios. Esta clase ya incluye toda la implementación necesaria y por defecto no tendremos que modificar nada. Pero si queremos podemos modificar esta clase para añadirle más métodos o relaciones con otras tablas, etc.

Laravel también incluye varios controladores (LoginController, RegisterController, ResetPasswordController y ForgotPasswordController) para la autenticación de usuarios, los cuales los puedes encontrar en el espacio de nombres App\Http\Controllers\Auth (y en la misma carpeta). LoginController y RegisterController incluyen métodos para ayudarnos en el proceso de autenticación, registro y cierre de sesión; mientras que ResetPasswordController y ForgotPasswordController contienen la lógica para ayudarnos en el proceso de restaurar una contraseña. Para la mayoría de aplicaciones con estos métodos será suficiente y no tendremos que añadir nada más.
Rutas

Por defecto Laravel no incluye las rutas para el control de usuarios. No obstante, al instalar Laravel/Breeze podemos observar que se añade una línea en el fichero routes/web.php:

require __DIR__.'/auth.php';

Esa línea incluye todas las rutas definidas en el fichero routes/auth.php, las cuales puedes comprobar con el comando


php artisan route:list

Como se puede ver estas rutas ya están enlazadas con los controladores y métodos que incorpora el propio Laravel.
Como la instalación de Laravel/Breeze ha regenerado el fichero routes/web.php:, debemos regresar a la versión anterior, añadiendo tan solo la línea


require __DIR__.'/auth.php';

Para ello, copiaremos la línea anterior y desde un terminal ejecutamos el siguiente comando git desde el directorio de videoclub:

Vistas

Al instalar Laravel/Breeze, también se generarán todas las vistas necesarias para realizar el login, registro y para recuperar la contraseña. Todas estas vistas las podremos encontrar en la carpeta resources/views/auth con los nombres login.blade.php para el formulario de login, register.blade.php , etcétera. Estos nombres y rutas son obligatorios ya que los controladores que incluye Laravel accederán a ellos, por lo que no deberemos cambiarlos.

Autenticación de un usuario

Una vez configurado todo el sistema, añadidas las rutas y las vistas para realizar el control de usuarios ya podemos utilizarlo. Si accedemos a la ruta login nos aparecerá la vista con el formulario de login, solicitando nuestro email y contraseña para acceder. El campo tipo checkbox llamado "remember" nos permitirá indicar si deseamos que la sesión permanezca abierta hasta que se cierre manualmente. Es decir, aunque se cierre el navegador y pasen varios días el usuario seguiría estando autorizado.

Si los datos introducidos son correctos se creará la sesión del usuario y se le redirigirá a la ruta "/dashboard". Si queremos cambiar esta ruta tenemos que definir la constante HOME en el fichero app/Providers/RouteServiceProvider.php, por ejemplo:

    public const HOME = '/catalog';

Registro de un usuario

Si accedemos a la ruta register nos aparecerá la vista con el formulario de registro, solicitándonos los campos nombre, email y contraseña. Al pulsar el botón de envío del formulario se llamará a la ruta register por POST y se almacenará el nuevo usuario en la base de datos.

Si no hemos añadido ningún campo más en la migración no tendremos que configurar nada más. Sin embargo si hemos añadido algún campo más a la tabla de usuarios tendremos que actualizar el controlador RegisteredUserController: validate y create. En la llamada a validate simplemente tendremos que añadir dicho campo al array de validaciones (solo en el caso que necesitemos validarlo). Y en la llamada al método create tendremos que añadir los campos adicionales que deseemos almacenar. El código de este método es el siguiente:

protected function create(array $data) {
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'phone' => $data['phone'],     // Campo añadido
        'password' => bcrypt($data['password']),
    ]);
}

Como podemos ver utiliza el modelo de datos User para crear el usuario y almacenar las variables que recibe en el array de datos $request. En este array de datos nos llegarán todos los valores de los campos del formulario, por lo tanto, si añadimos más campos al formulario y a la tabla de usuarios simplemente tendremos que añadirlos también en este método.

Es importante destacar que la contraseña se cifra usando el método bcrypt, por lo tanto las contraseñas se almacenaran cifradas en la base de datos. Este cifrado se basa en la clave hash que se general al crear un nuevo proyecto de Laravel (ver capítulo de "Instalación") y que se encuentra almacenada en el fichero .env en la variable APP_KEY. Es importante que este hash se haya establecido al inicio (que no esté vacío o se uno por defecto) y que además no se modifique una vez la aplicación se suba a producción.
Registro manual de un usuario

Si queremos añadir un usuario manualmente lo podemos hacer de forma normal usando el modelo User de Eloquent, con la única precaución de cifrar la contraseña que se va a almacenar. A continuación se incluye un ejemplo de una función que crea un nuevo usuario a partir de los parámetros de entrada recibidos de un formulario:

public function store(Request $request) {
    $user = new User;
    $user->name = $request->input('name');
    $user->email = $request->input('email');
    $user->password = bcrypt( $request->input('password') );
    $user->save();
}

Acceder a los datos del usuario autenticado

Una vez que el usuario está autenticado podemos acceder a los datos del mismo a través del método Auth::user(), por ejemplo:

$user = Auth::user();

Este método nos devolverá null en caso de que no esté autenticado. Si estamos seguros de que el usuario está autenticado (porque estamos en una ruta protegida) podremos acceder directamente a sus propiedades:

$email = Auth::user()->email;

    Importante: para utilizar la clase Auth tenemos que añadir el espacio de nombres use Illuminate\Support\Facades\Auth;, de otra forma nos aparecerá un error indicando que no puede encontrar la clase.

El usuario también se inyecta en los parámetros de entrada de la petición (en la clase Request). Por lo tanto, si en un método de un controlador usamos la inyección de dependencias también podremos acceder a los datos del usuario:

use Illuminate\Http\Request;

class ProfileController extends Controller {
    public function updateProfile(Request $request) {
        if ($request->user()) {
            $email = $request->user()->email;
        }
    }
}

Cerrar la sesión

Si accedemos a la ruta logout por POST se cerrará la sesión y se redirigirá a la ruta /. Todo esto lo hará automáticamente el método destroy del controlador AuthenticatedSessionController.

Para cerrar manualmente la sesión del usuario actualmente autenticado tenemos que utilizar el método:

Auth::logout();

Posteriormente podremos hacer una redirección a una página principal para usuarios no autenticados.

    Importante: para utilizar la clase Auth tenemos que añadir el espacio de nombres use Illuminate\Support\Facades\Auth;, de otra forma nos aparecerá un error indicando que no puede encontrar la clase.

Comprobar si un usuario está autenticado

Para comprobar si el usuario actual se ha autenticado en la aplicación podemos utilizar el método Auth::check() de la forma:

if (Auth::check()) {
    // El usuario está correctamente autenticado
}

Sin embargo, lo recomendable es utilizar Middleware (como veremos a continuación) para realizar esta comprobación antes de permitir el acceso a determinadas rutas.

    Importante: Recuerda que para utilizar la clase Auth tenemos que añadir el espacio de nombres use Illuminate\Support\Facades\Auth;, de otra forma nos aparecerá un error indicando que no puede encontrar la clase.

Proteger rutas

El sistema de autenticación de Laravel también incorpora una serie de filtros o Middleware (ver carpeta app/Http/Middleware y el fichero app/Http/Kernel.php) para comprobar que el usuario que accede a una determinada ruta o grupo de rutas esté autenticado. En concreto para proteger el acceso a rutas y solo permitir su visualización por usuarios correctamente autenticados usaremos el middleware \Illuminate\Auth\Middleware\Authenticate.php cuyo alias es auth. Para utilizar este middleware tenemos que editar el fichero routes/web.php y modificar las rutas que queramos proteger, por ejemplo:

// Para proteger una clausula:
Route::get('admin/catalog', function() {
    // Solo se permite el acceso a usuarios autenticados
})->middleware('auth');

// Para proteger una acción de un controlador:
Route::get('profile', [ProfileController::class, 'show'])->middleware('auth');

Si el usuario que accede no está validado se generará una excepción que le redirigirá a la ruta login. Si deseamos cambiar esta dirección tendremos que modificar el método que gestiona la excepción, el cual lo podremos encontrar en App\Exceptions\Handler@unauthenticated.

Si deseamos proteger el acceso a toda una zona de nuestro sitio web (por ejemplo la parte de administración o la gestión de un recurso), lo más cómodo es crear un grupo con todas esas rutas que utilice el middleware auth, por ejemplo:

Route::group(['middleware' => 'auth'], function() {
    Route::get('catalog', [CatalogController::class, 'getIndex']);
    Route::get('catalog/create', [CatalogController::class, 'getCreate']);
    // ...
});

5.3. Ejercicios

En los ejercicios de esta sección vamos a completar el proyecto del videoclub terminando el procesamiento de los formularios y añadiendo el sistema de autenticación de usuarios.
Ejercicio 1 - Migración de la tabla usuarios (0.5 puntos)

En primer lugar vamos a crear la tabla de la base de datos para almacenar los usuarios que tendrán acceso a la plataforma de gestión del videoclub.

Como hemos visto en la teoría, Laravel ya incluye una migración con el nombre create_users_table para la tabla users con todos los campos necesarios. Vamos a abrir esta migración y a comprobar que los campos incluidos coinciden con los de la siguiente tabla:
Campo	Tipo	Modificador
id 	Autoincremental 	
name 	String 	
email 	String 	unique
password 	String 	
remember_token 	Campo remember_token 	
timestamps 	Timestamps de Eloquent 	 

Comprueba también que en el método down de la migración se deshagan los cambios que se hacen en el método up, en este caso sería eliminar la tabla.

Por último usamos el comando de Artisan que añade las nuevas migraciones y comprobamos con PHPMyAdmin que la tabla se ha creado correctamente con todos campos indicados.
Ejercicio 2 - Seeder de usuarios (0.5 puntos)

Ahora vamos a proceder a rellenar la tabla users con los datos iniciales. Para esto editamos el fichero de semillas situado en database/seeds/DatabaseSeeder.php y seguiremos los siguientes pasos:

    Creamos un método privado (dentro de la misma clase) llamado seedUsers() que se tendrá que llamar desde el método run de la forma:

public function run() {
    // ... Llamada al seed del catálogo

    self::seedUsers();
    $this->command->info('Tabla usuarios inicializada con datos!');
}

    Dentro del nuevo método seedUsers() realizamos las siguientes acciones:
        En primer lugar borramos el contenido de la tabla users.
        Y a continuación creamos un par de usuarios de prueba. Recuerda que para guardar el password es necesario encriptarlo manualmente usando el método bcrypt (Revisa la sección "Registro de un usuario").

Por último tendremos que ejecutar el comando de Artisan que procesa las semillas. Una vez realizado esto comprobamos en PHPMyAdmin que se han añadido los usuarios a la tabla users.
Ejercicio 3 - Sistema de autenticación (1 punto)

En este ejercicio vamos a completar el sistema de autenticación. En primer lugar ejecuta los comandos de Artisan para generar todas las rutas y vistas necesarias para el control de usuarios con Laravel/Breeze.

composer require laravel/ui --dev
php artisan ui vue --auth

A continuación edita el fichero routes/web.php y realiza las siguientes acciones:

    Elimina (o comenta) las rutas de login y logout que habíamos añadido manualmente en los primeros ejercicios a fin de que se utilicen las nuevas rutas definidas por Laravel.
    Añade un middleware de tipo grupo que aplique el filtro auth para proteger todas las rutas del catálogo (menos la raíz / y las de autenticación).
    Revisa mediante el comando de Artisan php artisan route:list las nuevas rutas y que el filtro auth se aplique correctamente.

Modifica la redirección que se realiza tras el login o el registro de un usuario para que redirija a la ruta /catalog. Para esto tienes que modificar la constante HOME en el fichero app/Providers/RouteServiceProvider.php, (revisa el apartado "Autenticación de un usuario" de la teoría).
Ejercicio 4 - Adaptar el layout (1 punto)

Con el cambio a la versión 8 de Laravel, vamos a utilizar dicho layout para las vistas del catálogo, lo que nos permitirá, posteriormente, utilizar las pantallas para VueJS.
navbar

Para continuar con la estructura que teníamos en el layout resources/views/layouts/master.blade.php, vamos a extraer del resources/views/layouts/app.blade.php el contenido que hay entre las etiquetas <nav> y </nav> para llevarlo al partial resources/views/partials/navbar.blade.php, sin borrar el contenido original, ya que nuestra intención es adaptarlo.

En el lugar que ocupaban las etiquetas <nav> y </nav>, referencia al partial navbar, tal y como estaba en el layout master: @include('partials.navbar').

Del navbar original, nos interesa trasladar, al nuevo navbar, el contenido existente entre las etiquetas <ul class="navbar-nav mr-auto"> y </ul>. Como, este contenido únicamente debe estar visible si el usuario está autenticado, las meteremos dentro de etiquetas de blade @guest, @elseguest y @endguest.

El resto del contenido del navbar original, lo podemos borrar.
layout app

Modifica las vistas del directorio catalog, además de la de logout, para que, en lugar de utilizar el layout resources/views/layouts/master.blade.php, utilice el resources/views/layouts/app.blade.php.

Borra el ficheroresources/views/layouts/master.blade.php.

Para cambiar el título de la página y el que aparece en el navbar, lo mejor es configurar el nombre de la aplicación. Esto se puede hacer en 2 ficheros: .env o config/app.php. En nuestro caso, lo haremos en .env, sabiendo que si alguien se descarga nuestra aplicación, tendrá que volver a configurar ese nombre de la aplicación. Si lo hubiéramos configurado en config/app.php, aquel que se descargue nuestra aplicación, ya tendrá configurado el nombre.

Comprueba en este punto que el sistema de autenticación funciona correctamente: no te permite entrar a la rutas protegidas si no estás autenticado, puedes acceder con los usuarios definidos en el fichero de semillas y funciona el botón de cerrar sesión.
Ejercicio 5 - Añadir y editar películas (1 punto)

En primer lugar vamos a añadir las rutas que nos van a hacer falta para recoger los datos al enviar los formularios. Para esto editamos el fichero de rutas y añadimos dos rutas (también protegidas por el filtro auth):

    Una ruta de tipo POST para la url catalog/create que apuntará al método postCreate del controlador CatalogController.
    Y otra ruta tipo PUT para la url catalog/edit que apuntará al método putEdit del controlador CatalogController.

A continuación vamos a editar la vista catalog/edit.blade.php con los siguientes cambios:

    Revisar que el método de envío del formulario sea tipo PUT.
    Tenemos que modificar todos los inputs para que, como valor del campo, ponga el valor correspondiente de la película. Por ejemplo en el primer input tendríamos que añadir value="{{$pelicula->title}}". Realiza lo mismo para el resto de campos: year, director, poster y synopsis. El único campo distinto será el de synopsis ya que el input es tipo textarea, en este caso el valor lo tendremos que poner directamente entre la etiqueta de apertura y la de cierre.

Por último tenemos que actualizar el controlador CatalogController con los dos nuevos métodos. En ambos casos tenemos que usar la inyección de dependencias para añadir la clase Request como parámetro de entrada (revisa la sección "Datos de entrada" de la teoría). Además para cada método haremos:

    En el método postCreate creamos una nueva instancia del modelo Movie, asignamos el valor de todos los campos de entrada (title, year, director, poster y synopsis) y los guardamos. Por último, después de guardar, hacemos una redirección a la ruta /catalog.
    En el método putEdit buscamos la película con el identificador pasado por parámetro, actualizamos sus campos y los guardamos. Por último realizamos una redirección a la pantalla con la vista detalle de la película editada.

    Nota: de momento en caso de error no se mostrará nada.

