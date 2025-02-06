## Utilizando Laravel 11 para el desarrollo de un servicio web REST
Laravel 11 para el desarrollo de una api utiliza el módulo [laravel sanctum](https://documentacionlaravel.com/docs/11.x/sanctum) para la gestión de la autenticación del usuario

### Preparar el desarollo de la API

Para crear una api, ejecutamos el comando dentro del proyecto 

```bash
 php artisan install:api
```
Y vemos que nos instala `Laravel sanctum`, se crea un `fichero de rutas para las API` y nos ofrece ejecutar la migración donde ha creado una `nueva tabla personal_access_tokens`

Ahora debemos ir al `modelo User` y añadir `HasApiTokens`, esta configuración solo la requerimos si el sistema require autenticación.

```php

<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable, HasApiTokens;
```
Si ejecutamos 
```bash
 php artisan route:list
```
Nos muestra las rutas necesarias que nos ha creado la ruta **api/user**  y la ruta **sanctum/csrf-cookie**  que es para utilizarla en aplicaciones SPA (Una single-page application)

Vamos a utilizar [**Swagger**](https://swagger.io) para la documentación y prueba del servicio

Swagger es un conjunto de herramientas de software de código abierto que se utiliza para diseñar, construir, documentar y consumir servicios web RESTful. Swagger permite a los desarrolladores describir la estructura de sus APIs de manera estandarizada utilizando el formato OpenAPI Specification (OAS). Esto facilita la generación de documentación interactiva, la validación de las APIs y la creación de clientes y servidores en varios lenguajes de programación.

Swagger incluye herramientas como Swagger Editor, Swagger UI y Swagger Codegen, que ayudan en diferentes etapas del ciclo de vida de una API. Por ejemplo, Swagger UI proporciona una interfaz gráfica interactiva para probar y visualizar las APIs, mientras que Swagger Codegen permite generar código cliente y servidor a partir de la especificación de la API.

Instalamos swagger a través de composer:

```bash 
composer require darkaonline/l5-swagger 
php artisan vendor:publish --provider "L5Swagger\L5ServiceProvider"
```
y cada vez que realizamos un cambio en la documentación utilizamos el comando:

```bash 
php artisan L5-swagger:generate
```
Para ver la documentación en el servidor atiende a la ruta **api/documentation**

Ejemplo:

![swagger ejemplo](img/Swagger.jpg)


### Definir una API que no requiere autenticación

Las rutas de un servicio web las definimos en el fichero **api.php** que se encuentra en el directorio `routes`.

Si vamos a definir una API que implemente un CRUD, por ejemplo, una entidad `Producto` gestionada a través de un controlador denominado `ProductoController`:
```bash 
Route::apiResource('productos', ProductoController::class);
```
Nos genera todas las rutas necesarias para gestionar el CRUD

![rutas de apiresource](img/rutas.jpg)

Como vemos en la imagen nos genera todas las rutas necesarias para crear un CRUD en este ejemplo de la entidad `Producto`.

#### Ejemplo básico CRUD
Los pasos a realizar:
1. Crearemos un proyecto nuevo denominado `ApiProductos`
2.  Realizaremos los pasos de **Preparar el desarrollo de la api** del punto anterior
3. Para el ejemplo trabajaremos con dos entidades:
  * Crearemos una tabla denominada `categorias` y otra tabla denominada `productos`
  * Crearemos los modelos `Categoria` y `Producto`
 
```php
php artisan make:model Categoria
php artisan make:migration create_categorias_table

``` 
La tabla categorias en el método up tiene:
```php
  public function up(): void
    {
        Schema::create('categorias', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',100);
            $table->timestamps();
        });
    }
``` 
La tabla productos en el método up tiene:
```php
 public function up(): void
    {
        Schema::create('productos', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',100);
            $table->text('descripcion')->nullable();
            $table->decimal('precio', 8, 2);
            $table->integer('stock');
            $table->unsignedBigInteger('categoria_id')->nullable();
            $table->foreign('categoria_id')->references('id')->on('categorias');
            $table->timestamps();
        });
    }
```
El modelo de Categoria, que a continuación muestro figura documentado con swagger
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
/**
 * @OA\Schema(
 *  schema="Categoria",
 *  type="object",
 *  title="Categoria",
 *  @OA\Property(property="id", type="integer", example="1"),
 *  @OA\Property(property="nombre", type="string", example="Categoria 1")
 *  )
 */
class Categoria extends Model
{
    protected $table = 'categorias';
    protected $fillable = ['nombre'];
    public function productos()
    {
        return $this->hasMany(Producto::class);
    }
}
```
Realiza el modelo Producto y crea un conjunto de datos ficticios para ambas entidades a través de Seeder y Factory
Por ejemplo, para la entidad 'Categoria' crear un 'CategoriaSeeder'
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use App\Models\Categoria;

class CategoriaSeeder extends Seeder
{
   private $familia=[
        'Electrodomésticos',
        'Informática',
        'Telefonía',
        'Moda',
        'Deporte',
        'Hogar',
        'Jardín',
        'Bricolaje',
        'Mascotas',
        'Juguetes',
    ];
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        foreach ($this->familia as $familia) {
            Categoria::create(['nombre' => $familia]);
        }
    }
}
```
4. Una vez preparados los datos vamos a crear un `Resource` para cada entidad. Un `Resource` en Laravel se utiliza para transformar los modelos y colecciones de modelos en JSON, lo que es útil para estructurar las respuestas de la API.

Aquí tienes un ejemplo de cómo crear un `ProductoResource`:

**Crear el Resource**:
   Ejecuta el siguiente comando en la terminal para crear el resource:
   ```bash
   php artisan make:resource ProductoResource
   ```

**Definir el Resource**:
   Abre el archivo generado en `app/Http/Resources/ProductoResource.php` y define cómo se transformará el modelo `Producto` en JSON:
   ```php
   // filepath: /app/Http/Resources/ProductoResource.php
   <?php
    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class ProductoResource extends JsonResource
    {
        /**
        * Transform the resource into an array.
        *
        * @return array<string, mixed>
        */
        public function toArray(Request $request): array
        {
            return [
                'id' => $this->id,
                'nombre' => $this->nombre,
                'descripcion' => $this->descripcion,
                'precio' => $this->precio,
                'stock' => $this->stock,
                'categoria' =>new CategoriaResource($this->whenLoaded('categoria')),
            ];
        }
    }
   ```

**Usar el Resource en el Controlador**:
   En el controlador `ProductoController`, usa el `ProductoResource` para devolver las respuestas:
   ```php
   // filepath: /app/Http/Controllers/ProductoController.php
   use App\Http\Resources\ProductoResource;
   use App\Models\Producto;

   class ProductoController extends Controller
   {
       public function index()
       {
           $productos = Producto::with('categoria')->get();
           
           return ProductoResource::collection($productos);
       }

       public function show($id)
       {
           $producto = Producto::with('categoria')->findOrFail($id);
           return new ProductoResource($producto);
       }

       // Otros métodos del controlador...
   }
   ```

Esto asegura que las respuestas de la API estén estructuradas de manera consistente y clara.

5. Definir el controlador `ProductoController` y documentar el controlador y los métodos para realizar la pruebas de la API a través de Swagger
 
Por ejemplo, como documentación del controlador `ProductoController` debe realizarse antes de la definición de la clase
```php 
    <?php
    namespace App\Http\Controllers;

    use App\Http\Resources\ProductoResource;
    use Illuminate\Http\Request;
    use App\Models\Producto;
    /**
     * @OA\Info(title="API Productos", version="1.0",description="API de productos",
     * @OA\Server(url="http://localhost:8000"),
     * @OA\Contact(email="email@gmail.com"))
     */
    class ProductoController extends Controller
    {
```
Y un ejemplo de documentación del método `show` del controlador `ProductoController` 
```php 
   /**
     * Display the specified resource.
     */
    /**
     * @OA\Get(
     *  path="/api/productos/{id}",
     *  summary="Obtener un producto",
     *  description="Obtener un producto por su id",
     *  operationId="show",
     *  tags={"productos"},
     *  @OA\Parameter(
     *      name="id",
     *      in="path",
     *      description="Id del producto",
     *   required=true,
     *   @OA\Schema(type="integer",example="1")
     *  ),
     *  @OA\Response(
     *  response=200,
     *  description="Producto encontrado",
     *  @OA\JsonContent(ref="#/components/schemas/Producto")
     * ),
     *  @OA\Response(
     *  response=404,
     *  description="Producto no encontrado"
     *  )
     * )
     */
    public function show(Producto $producto)
    {
       return new ProductoResource($producto->load('categoria'));
    }
```
6. Cuando realizamos una Api CRUD, es importante realizar una validación de los datos creando un `Request Especifico` o bien realizando la validación con el `método validate` de la clase `Request`
Por ejemplo, si creamos un producto deberiamos validar:

```php
        $validatedData = $request->validate([
            'nombre' => 'required|max:100|unique:productos',
            'descripcion' => 'nullable|string',
            'precio' => 'required|numeric|min:0',
            'stock' => 'required|integer|min:0',
            'categoria_id' => 'nullable|integer|exists:categorias,id'
        ],[
            'nombre.max'=>'El nombre no puede tener más de 100 caracteres',
            'nombre.required'=>'El nombre es obligatorio',
            'nombre.unique'=>'El nombre ya existe',
            'precio.required'=>'El precio es obligatorio',
            'precio.numeric'=>'El precio debe ser un número',
            'precio.min'=>'El precio no puede ser negativo',
            'stock.required'=>'El stock es obligatorio',
            'stock.min'=>'El stock no puede ser negativo',
            'stock.integer'=>'El stock debe ser un número entero',
            'categoria_id.exists'=>'La categoría no existe'
        ]);
```
:computer: Hoja07_WebServices_01

### Definir una API que requiere autenticación
Laravel utiliza tokens de autenticación para asegurar las APIs. El paquete Laravel Sanctum proporciona un sistema de autenticación de API ligero para SPAs (Single Page Applications), aplicaciones móviles y simples APIs basadas en tokens.

#### Cómo funciona:
1. **Generación de Tokens**: Los usuarios autenticados pueden generar tokens de acceso que se almacenan en la base de datos.
2. **Uso de Tokens**: Estos tokens se envían con las solicitudes API en los encabezados de autorización.
3. **Validación de Tokens**: Laravel valida estos tokens para asegurar que las solicitudes provienen de usuarios autenticados.

Laravel Sanctum genera tokens de acceso que se pueden usar como `Bearer Tokens` en las solicitudes API. `Un Bearer Token` es un tipo de token de acceso que se incluye en el encabezado de autorización de una solicitud HTTP.

En el paso **preparar el desarrollo de la api** hemos establecido que el usuario puede generar tokens de autotenticación, pero tenemos que realizar algunas configuraciones más que explicamos a continuación:

#### Preparar Swagger para api autenticada
Para que Swagger implemente autenticación es necesario instalar otra libreria
```bash
   composer require zircote/swagger-php
```
Debemos también documentar el controlador en nuestro ejemplo productos con `@OA\SecurityScheme`
```
 * @OA\SecurityScheme(
 *     type="http",
 *     description="Use a token to authenticate",
 *     name="Authorization",
 *     in="header",
 *     scheme="bearer",
 *     bearerFormat="JWT",
 *     securityScheme="bearerAuth",
 * )
 ```
 y también documentar cada uno de los métodos que requieran seguridad por ejemplo el store (crear un producto) donde antes del `@OA\RequestBody` añadimos la etiqueta security 
```
 * security={{"bearerAuth":{}}},
 ```
También añadiremos la respuesta de no autorizado en el caso que no tenga toquen asociado.
```
* @OA\Response(
    * response=401,
    * description="No autorizado"
* )
```
En Swagger aparece un botón en la cabecera denominado `Autorize` que es donde introduciremos el token que nos devuelve el login del usuario. 
Tenemos que crear un login de usuario donde al identificarse si es un usuario del sistema genere el token requerido para realizar las acciones que requieran seguridad. Como paso previo debemos tener crear al menos un usuario en la aplicación.

#### Ejemplo de Login

Definimos un controlador `LoginController`  donde comprobamos por correo electrónico y password que sea un usuario de la aplicación en ese caso devolvemos el nombre, correo electrónico y el token.
```php
class LoginController extends Controller
{
    /**
     * Handle the incoming request.
     */

    /**
     * @OA\Post(
        * path="/api/login",
        * summary="Login",
        * description="Login del usuario",
        * operationId="login",
        * tags={"login"},
        * @OA\RequestBody(
        *    required=true,
        *    description="Datos del usuario",
        *    @OA\JsonContent(
        *       required={"email","password"},
        *       @OA\Property(property="email", type="string", format="email", example="prueba@prueba.es"),
        *       @OA\Property(property="password", type="string", format="password", example="12345678")
        *    ),
        * ),
        * @OA\Response(
        *  response=200,
        *  description="Login correcto",
        *  @OA\JsonContent(
        *       @OA\Property(property="user", type="object",
        *           @OA\Property(property="name", type="string"),
        *           @OA\Property(property="email", type="string")
        *       ),
        *       @OA\Property(property="token", type="string")
        *  )
        * ),
        * @OA\Response(
        *  response=401,
        *  description="No autorizado",
        *  @OA\JsonContent(
        *       @OA\Property(property="message", type="string")
        *  )
        * )
        * )
    */
    public function __invoke(Request $request)
    {
        $user=User::where('email',$request->email)->first();
        if(!$user || !Hash::check($request->password,$user->password)){
            return response()->json(['message'=>'No autorizado'],401);
         }
        return response()->json([
        'user'=>[
            'name'=>$user->name,
            'email'=>$user->email,
        ],
        'token'=>$user->createToken($request->email)->plainTextToken]);
    }
}
```
Creamos la ruta de login de tipo post en el fichero de rutas de apis

```php
Route::post('/login',LoginController::class);
```
Como ejemplo vamos a poner seguridad a la creación del usuario, por lo que tendremos que modificar la documentación del método según las ins
```
/**
*@OA\Post(
    * path="/api/productos",
    * summary="Crear un producto",
    * description="Crear un producto",
    * operationId="store",
    * tags={"productos"},
    * security={{"bearerAuth":{}}},
    * @OA\RequestBody(
        * required=true,
        * description="Datos del producto",
        * @OA\JsonContent(
            * required={"nombre","precio","stock"},
            * @OA\Property(property="nombre", type="string", example="Producto 1"),
            * @OA\Property(property="descripcion", type="string", example="Descripción del producto"),
            * @OA\Property(property="precio", type="number", format="float", example="10.5"),
            * @OA\Property(property="stock", type="integer", example="10"),
            * @OA\Property(property="categoria_id", type="integer", example="1")
        *),

    *),
    * @OA\Response(
        * response=201,
        * description="Producto creado",
        * @OA\JsonContent(ref="#/components/schemas/Producto")
    *),
    * @OA\Response(
        * response=422,
        * description="Error de validación"
    * ),
    * @OA\Response(
        * response=401,
        * description="No autorizado"
    * )
* )
*/
```

```php
Route::apiResource('productos', ProductoController::class)->except('store');
Route::middleware('auth:sanctum')->post('productos', [ProductoController::class, 'store']);
```
:computer: Hoja07_WebServices_02











