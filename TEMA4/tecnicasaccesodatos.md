# UT4 UTILIZACIÓN DE TÉCNICAS DE ACCESO A DATOS

## Índice
- [UT4 UTILIZACIÓN DE TÉCNICAS DE ACCESO A DATOS](#ut4-utilización-de-técnicas-de-acceso-a-datos)
  - [Índice](#índice)
  - [Acceso a bases de datos desde PHP](#acceso-a-bases-de-datos-desde-php)
  - [MySQL /MariaDB](#mysql-mariadb)
  - [MySQLi](#mysqli)
  - [PHP Data Objects(PDO)](#php-data-objectspdo)
    - [PDO: establecimiento de conexiones](#pdo-establecimiento-de-conexiones)
      - [Las variables figuren en un archivo .env](#las-variables-figuren-en-un-archivo-env)
    - [PDO: ejecución de consultas](#pdo-ejecución-de-consultas)
    - [PDO: transacciones](#pdo-transacciones)
    - [PDO: obtención y utilización de conjuntos de resultados](#pdo-obtención-y-utilización-de-conjuntos-de-resultados)
    - [PDO: consultas preparadas](#pdo-consultas-preparadas)
  - [Errores y manejo de excepciones](#errores-y-manejo-de-excepciones)
    - [¿Qué es un error?](#qué-es-un-error)
    - [¿Qué es una excepción?](#qué-es-una-excepción)
      - [¿Cómo se manejan las excepciones?](#cómo-se-manejan-las-excepciones)
        - [try](#try)
        - [catch](#catch)
        - [finally](#finally)
        - [throw](#throw)
      - [Creando nuestras excepciones](#creando-nuestras-excepciones)

## Acceso a bases de datos desde PHP

PHP soporta más de 15 **sistemas gestores de bases de datos**: SQLite, Oracle, SQL Server, PostgreSQL, IBM DB2, MySQL, etc
Hasta la versión 5 de PHP, el acceso a las bases de datos se hacía principalmente utilizando extensiones específicas para cada sistema gestor de base de datos (**extensiones nativas**).

A partir de la versión 5 de PHP se introdujo en el lenguaje una extensión para acceder de una forma común a distintos sistemas gestores: **PDO**.

La gran ventaja de PDO es que podemos seguir utilizando una misma sintaxis aunque cambiemos el motor de nuestra base de datos.

Mientras PDO ofrece un conjunto común de funciones, las extensiones nativas normalmente ofrecen más potencia (acceso a funciones específicas de cada gestor de base de datos) y en algunos casos también mayor velocidad.

## MySQL /MariaDB

**MySQL** es un sistema de gestión de bases de datos relacional desarrollado bajo **licencia dual**: Licencia pública general/Licencia comercial por Oracle Corporation y está considerada como la base de datos de código abierto más popular del mundo,​​ y una de las más populares en general junto a Oracle y Microsoft SQL Server.

![Logo de MySQL](img/mysql.png)

**MariaDB** es un sistema de gestión de bases de datos derivado de MySQL con licencia GPL (General Public License). Es desarrollado por Michael (Monty) Widenius —fundador de MySQL—, la fundación MariaDB y la comunidad de desarrolladores de software libre.

![Logo de MariaDB](img/mariaDB.png)

Ambas bases de datos incorporan múltiples motores de almacenamiento siendo alguno de ellos:

**MyISAM** (**Aria** en MariaDB):. Muy rápido pero a cambio no contempla integridad referencial ni tablas transaccionales. 

**InnoDB** (**XtraDB** en MariaDB):  motor que se utiliza por defecto, es un poco más lento pero sí soporta tanto integridad referencial como tablas transaccionales.

## MySQLi

Esta extensión se desarrolló para aprovechar las ventajas que ofrecen las versiones 4.1.3 y posteriores de MySQL, y viene incluida con PHP a partir de la versión 5. 

Ofrece un interface de programación dual, pudiendo accederse utilizando objetos o funciones.
Por ejemplo, para establecer una conexión con un servidor MySQL y consultar su versión, podemos utilizar cualquiera de las siguientes formas:

- Utilizando objetos
```php
//utilizando constructores y métodos de la programación orientada a objetos
$conexion = new mysqli('localhost', 'usuario', 'contraseña', 'base_de_datos');
print $conexion‐>server_info;
```
- Utilizando funciones 
```php 
// utilizando llamadas a funciones 
$conexion = mysqli_connect('localhost', 'usuario', 'contraseña', 'base_de_datos');
print mysqli_get_server_info($conexion);
```

## PHP Data Objects(PDO)

PDO se basa en las características de orientación a objetos de PHP pero, al contrario que la extensión MySQLi, no ofrece un interface de programación dual. 

Para acceder a las funcionalidades de la extensión hay que emplear los objetos que ofrece, con sus métodos y propiedades. No existen funciones alternativas.

### PDO: establecimiento de conexiones
Para establecer una conexión con una base de datos utilizando PDO hay que instanciar un objeto de la clase `PDO` pasándole los siguientes parámetros (solo el primero es obligatorio):

* Origen de datos (DSN). Es una cadena de texto que indica qué controlador se va a utilizar y a continuación, separadas por el carácter dos puntos, los parámetros específicos necesarios por el controlador, como por ejemplo el nombre o dirección IP del servidor y el nombre de la base de datos.
* Nombre de usuario con permisos para establecer la conexión.
* Contraseña del usuario.
* Opciones de conexión, almacenadas en forma de array.

```php 
$dwes = new PDO('mysql:host=localhost;dbname=dwes', 'dwes', 'abc123.');
```

Si se utiliza **el controlador para MySQL**, los parámetros específicos para utilizar en la cadena DSN (separadas unas de otras por el carácter punto y coma) a continuación del prefijo mysql: son los siguientes:
- **host**: nombre o dirección IP del servidor.
- **port**: número de puerto TCP en el que escucha el servidor.
- **dbname**: nombre de la base de datos.
- **unix_socket**: socket de MySQL en sistemas Linux.

Para indicar que utilice codificación UTF-8 para los datos que se transmitan:

```php 
$opciones = array(PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8"); 
$dwes = new PDO('mysql:host=localhost;dbname=dwes', 'dwes', 'abc123.', $opciones);
```
Ejemplo de conexión controlando las excepciones:

```php 
 try{
     $connection= new PDO(
        dsn: 'mysql:host=localhost;dbname=dwes;charset=utf8mb4',
        username: 'dwes',
        password:'abc123.',
     )
     echo 'Conexión establecida correctamente';
 }catch (PDOException $e){
    echo match($e->getCode()){
        1049 => 'Base de datos no encontrada',
        1045 => 'Acceso denegado',
        2002 => 'Conexión rechazada',
        default => 'Error desconocido',
    };
}
```
####  Las variables figuren en un archivo .env

A través de la librería externa [phpdotenv](https://github.com/vlucas/phpdotenv) que nos permite definir un fichero `.env` para tener todas las variables de entorno definidas.

Un ejemplo del archivo **.env** 

```shell

DB_DSN=mysql:host=localhost:3306;dbname=prueba;charset=utf8mb4
DB_USERNAME=root
DB_PASSWORD=
```
Un ejemplo de la clase de conexión

```php
require_once __DIR__ . '/vendor/autoload.php';

$dotenv = \Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

final class ConnectionPDODotenv
{
    private static ?PDO $connection = null;

    final private function __construct() {}

    final public static function getConnection(): ?PDO
    {
        try {
            if ( ! self::$connection) {
                $opciones = array(PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8");
                self::$connection = new PDO(
                    dsn: $_ENV['DB_DSN'],
                    username: $_ENV['DB_USERNAME'],
                    password: $_ENV['DB_PASSWORD'],
                    options:$opciones
                );
                 self::$connection->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_EXCEPTION);
            }
        } catch (PDOException $e) {
            echo match ($e->getCode()) {
                1049 => 'Base de datos no encontrada',
                1045 => 'Acceso denegado',
                2002 => 'Conexión rechazada',
                default => 'Error desconocido',
            };
        }

        return self::$connection;
    }
     private function __clone() {}
}

$connection = ConnectionPDODotenv::getConnection();

if ($connection instanceof PDO)
{
    echo 'Conexión establecida correctamente';
}

``` 

:computer: Hoja04_BBDD_01 (Ej 1,2,3,4)

### PDO: ejecución de consultas
Hay que diferenciar entre las sentencias SQL que no devuelven datos, de aquellas que sí lo devuelven.

En el caso de las consultas de acción, como INSERT, DELETE o UPDATE, el **método exec** devuelve el número de registros afectados.

```php 
$registros=$dwes->exec('DELETE FROM stock WHERE unidades=0');
echo "<p>Se han borrado $registros registros.</p>";
```
Si la consulta genera un conjunto de datos (SELECT) hay que utilizar el **método query**, que devuelve un objeto de la clase **PDOStatement**.

```php 
$dwes = new PDO("mysql:host=localhost;dbname=dwes", "dwes", "abc123."); 
$resultado = $dwes‐>query("SELECT producto, unidades FROM stock"); 
```
### PDO: transacciones

Por defecto PDO trabaja en modo "autocommit", es decir, que confirma de forma automática cada sentencia que ejecuta el servidor. Para trabajar con transacciones, PDO incorpora tres métodos:
- **beginTransaction**: deshabilita el modo "autocommit" y comienza una nueva transacción, que finalizará cuando se ejecute uno de los dos métodos siguientes.
- **commit**: confirma la transacción actual.
- **rollback**: revierte los cambios llevados a cabo en la transacción actual.
Una vez ejecutado un commit o un rollback, se volverá al modo de confirmación automática.

```php 
$ok = true; 
$dwes->beginTransaction(); 
if($dwes->exec('DELETE …') == 0)  $ok = false; 
if($dwes->exec('UPDATE …') == 0)  $ok = false; 
… 
if ($ok) $dwes->commit();  // Si todo fue bien confirma los cambios 
else $dwes->rollback(); // y si no, los revierte
```
### PDO: obtención y utilización de conjuntos de resultados

Al igual que en MySQLi, en PDO hay varias posibilidades para tratar con el conjunto de resultados devueltos por el método query. La más utilizada es el **método fetch**:

```php 
$resultado = $dwes->query('SELECT campo1, campo2 FROM tabla WHERE condicion');
while ($registro = $resultado->fetch()) 
{
echo $registro['campo1'].": ".$registro['campo2']."<br />";
} 
```
Por defecto, el **método fetch** genera y devuelve, a partir de cada registro, un array con claves numéricas y asociativas. Para cambiar su comportamiento, admite un parámetro opcional que puede tomar uno de los siguientes valores:
- **PDO::FETCH_ASSOC**: devuelve solo un array asociativo.
- **PDO::FETCH_NUM**: devuelve solo un array con claves numéricas.
- **PDO::FETCH_BOTH**: devuelve un array con claves numéricas y asociativas. Es el comportamiento por defecto.
- **PDO::FETCH_OBJ**: devuelve un objeto cuyas propiedades se corresponden con los campos del registro.

```php 
$resultado = $dwes->query('SELECT campo1, campo2 FROM tabla WHERE condicion');
while ($registro = $resultado->fetch(PDO::FETCH_OBJ)) 
{
	echo $registro->campo1.": ".$registro->campo2."<br />";
} 
```
- **PDO::FETCH_LAZY**: devuelve tanto el objeto como el array con clave dual anterior.
- **PDO::FETCH_BOUND**: devuelve true y asigna los valores del registro a variables, según se indique con el `método bindColumn`. Este método debe ser llamado una vez por cada columna, indicando en cada llamada el número de columna (empezando en 1) y la variable a asignar.

```php
$resultado = $dwes->query('SELECT campo1, campo2 FROM tabla WHERE condicion');
$resultado->bindColumn(1, $campo1);
$resultado->bindColumn(2, $campo2);
while ($registro = $resultado->fetch(PDO::FETCH_BOUND)) 
{
	echo $campo1.": ".$campo2."<br />";
} 
```
:computer: Hoja04_BBDD_01(Ej 5,6,7 y 8)

### PDO: consultas preparadas

Para preparar la consulta se utiliza el **método prepare** de la clase PDO. Este método devuelve un objeto de la clase **PDOStatement**. Los parámetros se pueden marcar utilizando signos de interrogación.

```php
$consulta=$dwes->prepare('INSERT INTO tabla (campo1, campo2) VALUES (?, ?)');
```
O también se pueden utilizar parámetros con nombre, precediéndolos por el símbolo de dos puntos:

```php
$consulta=$dwes->prepare('INSERT INTO tabla (campo1, campo2) VALUES (:cm1, :cm2)’);
```
Si se utiliza parámetros con nombre hay que indicar ese nombre en la llamada al método `bindParam`

```php
$campo1 = "valor_campo1";
$campo2 = "valor_campo2";
$consulta->bindParam(1, $campo1); //o $consulta->bindParam(:cm1, $campo1);
$consulta->bindParam(2, $campo2); //o $consulta->bindParam(:cm2, $campo2);
```
Una vez preparada la consulta y enlazados los parámetros con sus valores, se ejecuta la consulta utilizando el **método execute**.

```php
$consulta->execute();
```
:computer: Hoja04_BBDD_01(Ej 9)

## Errores y manejo de excepciones

### ¿Qué es un error?
Un error es un problema que ocurre en tiempo de ejecución y que interrumpe el flujo normal de ejecución del programa.


### ¿Qué es una excepción?
A partir de la versión 5 se introdujo en PHP un modelo de excepciones similar al existente en otros lenguajes de programación.
Una excepción es un error que ocurre en tiempo de ejecución y que interrumpe el flujo normal de ejecución del programa. Las excepciones son objetos que representan errores. Cuando ocurre un error, el intérprete de PHP crea un objeto de la __clase Exception__ o de una __clase derivada de Exception__ y lo lanza. El código que llama a la función que generó el error puede capturar la excepción y manejarla de forma adecuada.

#### ¿Cómo se manejan las excepciones?
Las excepciones se manejan con la sentencia try/catch. La sentencia try/catch permite definir un bloque de código que se va a intentar ejecutar y un bloque de código que se va a ejecutar si se produce una excepción. La sentencia try/catch se puede usar para capturar excepciones lanzadas por cualquier código PHP, incluyendo código de terceros.

- El código susceptible de producir algún error se introduce en un bloque **try**.
- Cuando se produce algún error, se lanza una excepción utilizando la instrucción **throw**.
- Después del bloque try debe haber como mínimo un bloque **catch** encargado de procesar el error.
- Si una vez acabado el bloque try no se ha lanzado ninguna excepción, se continúa con la ejecución en la línea siguiente al bloque o bloques catch.


##### try 
La sentencia try define un bloque de código que se va a intentar ejecutar. Si se produce una excepción, el código que sigue a la sentencia try no se ejecuta y el intérprete PHP busca una sentencia catch que pueda manejar la excepción.

##### catch
La sentencia catch define un bloque de código que se va a ejecutar si se produce una excepción. La sentencia catch debe especificar el tipo de excepción que se va a manejar. Si se produce una excepción que no es del tipo especificado, la sentencia catch no se ejecuta y la excepción se propaga al siguiente bloque catch más externo.

##### finally
La sentencia finally define un bloque de código que se va a ejecutar siempre, independientemente de si se produce una excepción o no. La sentencia finally se ejecuta después de que se ejecuta el bloque try y después de que se ejecuta el bloque catch.

##### throw
La sentencia throw permite lanzar una excepción. La sentencia throw debe especificar el tipo de excepción que se va a lanzar. Si se lanza una excepción que no es del tipo especificado, la excepción se propaga al siguiente bloque catch más externo.

Por ejemplo, para lanzar una excepción cuando se produce una división por cero podrías hacer: 

```php
function divide($dividendo, $divisor): int|float
{
    if ($divisor === 0) {
        throw new Exception('No se puede dividir por cero');
    }

    if (!is_int($dividendo) || !is_int($divisor)) {
        throw new InvalidArgumentException('Los argumentos deben ser números enteros');
    }

    return $dividendo / $divisor;
}

try {
    echo divide(100, 0.5);
} catch (InvalidArgumentException $exception) {
    echo 'InvalidArgumentException: ' . $exception->getMessage();
} catch (Exception $exception) {
    echo 'Exception: ' . $exception->getMessage();
}

```
#### Creando nuestras excepciones
Ejemplo:
```php
class InvalidProductException extends Exception {} // creo una nueva excepcion InvalidProductExcepcion

final class Cart {   
    private array $items = [];

    public function getItems(): array {
        return $this->items;
    }

    public function addProduct(array $product): void {
        if (!isset($product['name']) || !isset($product['price'])) {
            throw new InvalidProductException('El producto no es válido');
        }

        $this->items[] = $product;
    }
}

$cart = new Cart;

try {
    $cart->addProduct([
        'name' => 'Producto 1',
        'precio' => 10, //equivocacion debería ser price
    ]);
    var_dump($cart->getItems());
} catch (InvalidProductException $ex) {
    echo $ex->getMessage();
}
```
En este ejemplo saldrá por la excepción debido a que el nombre es `price`  y por equivocación ha puesto `precio`.

PHP define una **clasificación de los errores** que se pueden producir en la ejecución de un programa y ofrece métodos para ajustar el tratamiento de los mismos. Para hacer referencia a cada uno de los niveles de error, PHP define una serie de **constantes**. 

Cada nivel se identifica por una constante. Por ejemplo, la constante `E_NOTICE` hace referencia a avisos que pueden indicar un error al ejecutar el script, y la constante `E_ERROR` engloba errores fatales que provocan que se interrumpa forzosamente la ejecución.

La configuración inicial de cómo se va a tratar cada error según su nivel se realiza en `php.ini` el fichero de configuración de PHP. 

Entre los principales parámetros que puedes ajustar están:
- **error_reporting**: indica qué tipos de errores se notificarán. Su valor se forma utilizando los operadores a nivel de bit para combinar las constantes anteriores. Su valor predeterminado es E_ALL & ~E_NOTICE que indica que se notifiquen todos los errores (E_ALL) salvo los avisos en tiempo de ejecución (E_NOTICE).
- **display_errors**: en su valor por defecto (On), hace que los mensajes se envíen a la salida estándar (y por lo tanto se muestren en el navegador). Se debe desactivar (Off) en los servidores que no se usan para desarrollo sino para producción.

`Desde código` se puede usar la función **error_reporting** con las constantes anteriores para establecer el nivel de notificación en un momento determinado.

Por ejemplo, en un entorno de desarrollo normalmente interesa que tengamos información de todos los errores  e incluso crear un fichero con los errores detectados:

```php
error_reporting(E_ALL | E_STRICT); // recoge todos los errores
ini_set('display_errors', 1); // es como si cambiase este paramétros en el fichero de configuración de php, activando que los mensajes se envién por la salida estándar
ini_set('log_errors', 1); // activo que se guarden los errores en un fichero de log
ini_set('error_log', __DIR__ . '/logs/php_errors.log'); // indico la ruta en donde guardar los errores.

```

La clase PDO permite definir la fórmula que usará cuando se produzca un error, utilizando el atributo `PDO::ATTR_ERRMODE`. Las posibilidades son:

- **PDO::ERRMODE_SILENT**: no se hace nada cuando ocurre un error. Es el comportamiento por defecto.
- **PDO::ERRMODE_WARNING**: genera un error de tipo `E_WARNING` cuando se produce un error.
- **PDO::ERRMODE_EXCEPTION**: cuando se produce un error lanza una excepción utilizando el manejador propio**PDOException**.

Es decir, que si quieres utilizar `excepciones` con la extensión PDO, debes configurar la conexión haciendo:

```php
$dwes->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION); 
```
Por ejemplo, el siguiente código captura la excepción que lanza PDO debido a que la tabla no existe: 

```php
$dwes = new PDO("mysql:host=localhost; dbname=dwes", "dwes", "abc123."); 
$dwes->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION); 
try { 
	$sql = "SELECT * FROM tablaincorrecta"; 
	$result = $dwes->query($sql); 
	… 
} 
catch (PDOException $p) { 
	echo "Error ".$p->getMessage()."<br />"; 
}
```

:computer: Hoja04_BBDD_02

:computer: Hoja04_BBDD_03

:computer: Hoja04_BBDD_04

:computer: Hoja04_BBDD_05

:computer: Hoja04_BBDD_06































