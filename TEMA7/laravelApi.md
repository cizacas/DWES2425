# Crear una API REST Laravel 11
Vamos a desarrollar una API rest, de gestión de autenticación utilizando [laravel sanctum](https://documentacionlaravel.com/docs/11.x/sanctum) y [postman](https://www.postman.com/) para pruebas y documentación de la API.

## Crear el proyecto
Creamos un proyecto en laravel

```
laravel new  api_ejemplo
```
## Preparar el desarollo de la API

Para crea una api, ejecutamos el comando dentro del proyecto creado 
```php
php artisan install:api
```
Y vemos que nos instala `Laravel sanctum`, se crea un `fichero de rutas para las API` y nos ofrece ejecutar la migración donde ha creado una `nueva tabla personal_access_tokens`

Ahora debemos ir al `modelo User` y añadir `HasApiTokens`

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
```php
php artisan route:list
```
Nos muestra las rutas necesarias que nos ha creado la ruta **api/user**  y la ruta **sanctum/csrf-cookie**  que es para utilizarla en aplicaciones SPA (Una single-page application)

/**
instalaremos la dependencia de ray de laravel spatie como dependencia de desarrollo
```php
composer require spatie/laravel-ray --dev
php artisan ray:publish-config
**/


## Definir el sistema de autenticación
Crear **un servicio** donde estén recogidas todas las posibles respuestas del Json y nos ayudamos delos [códigos de respuesta Http](https://developer.mozilla.org/es/docs/Web/HTTP/Status). Este servicio lo vamos a utilizar en todos los controladores para devolver la respuesta 

```php
php artisan make:class Services\API\Auth\ApiResponseService
```
```php
<?php
namespace App\Services\API\Auth;
use Illuminate\Http\JsonResponse;
use Symfony\Component\HttpFoundation\Response;

class ApiResponseService
{
    public static function success($data, $message = 'Success', $code = Response::HTTP_OK): JsonResponse
    {
        return response()->json([
            'status' => 'success',
            'message' => $message,
            'data' => $data,
        ], $code);
    }

    public static function error($message = 'Error', $code = Response::HTTP_BAD_REQUEST): JsonResponse
    {
        return response()->json([
            'status' => 'error',
            'message' => $message,
        ], $code);
    }

    public static function unauthorized($message = 'Unauthorized'): JsonResponse
    {
        return response()->json([
            'status' => 'error',
            'message' => $message,
        ], Response::HTTP_UNAUTHORIZED);
    }

    public static function forbidden($message = 'Forbidden'): JsonResponse
    {
        return response()->json([
            'status' => 'error',
            'message' => $message,
        ], Response::HTTP_FORBIDDEN);
    }

    public static function notFound($message = 'Not Found'): JsonResponse
    {
        return response()->json([
            'status' => 'error',
            'message' => $message,
        ], Response::HTTP_NOT_FOUND);
    }
}
```

Crear **una interfaz** para implementar el sistema de autenticación que tendrá al menos definición de los métodos login y logout

```php
php artisan make:interface Contracts\API\Auth\AuthServiceInterface
```
y en el implementamos los métodos para que nos devuelvan un `JsonResponse`

```php
<?php

namespace App\Contracts\API\Auth;

use Illuminate\Http\JsonResponse;

interface AuthServiceInterface
{
    public function login(array $credentials): JsonResponse;

    public function logout(): JsonResponse;
}
```
Crear **un servicio** que implementa la interfaz definida
```php
php artisan make:class Services\API\Auth\AuthSanctumService
```
Vamos a validar los datos de entrada al login por lo que vamos a crear un `Request`
```php
php artisan make:request API\Auth\LoginRequest
```
```php
<?php

namespace App\Http\Requests\API\Auth;

use Illuminate\Foundation\Http\FormRequest;

class LoginRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
     */
    public function rules(): array
    {
        return [
            'email' => 'required'|'email',
            'password' => 'required'|'string',
            'name' => 'required'|'string',
        ];
    }
}
```
Ahora vamos a crear los controladores que van a ser invocables para tener el código más segmentado aunque podriamos crear un solo controlador con todas las acciones.
```php
php artisan make:controller API\Auth\LoginController
php artisan make:controller API\Auth\LogoutController
```



## Ejemplo
Vamos a crear una librería con autores, generos y libros, donde un libro tendrá un autor y pertenecerá a un género Crearemos los modelos y la entidad asociada en base de datos. Para autores  y generos y en la definición de la tabla incorporamos el campo nombre con una longitud de 100 caracteres.
```php
php artisan make:model Autor
php artisan make:migration create_autores_table

```
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('autores', function (Blueprint $table) {
            $table->id();
            $table->string('nombre',100);
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('autores');
    }
};
```
Definiremos el libro con los campos que figuran a continuación y establecemos las relaciones con autores y generos
```php
php artisan make:model Libro -m
```
```php

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('libros', function (Blueprint $table) {
            $table->id();
            $table->foreignIdFor(\App\Models\Autor::class)->constrained();
            $table->foreignIdFor(\App\Models\Genero::class)->constrained();
            $table->string('titulo', 100);
            $table->string('isbn', 13);
            $table->integer('paginas');
            $table->unsignedTinyInteger('stock');
            $table->date('publicado_en');
            $table->timestamps();
        });
    }
```
Ahora configuraremos los modelos, con el nombre de la tabla y las relaciones establecidas. A continuación figura el ejemplo de Autor  y del Libro
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Autor extends Model
{
    protected $table = 'autores';
    protected $fillable = ['nombre'];

    public function libros():HasMany
    {
        return $this->hasMany(Libro::class);
    }
}

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Libro extends Model
{
    protected $fillable = ['titulo','isbn','autor_id','genero_id', 'paginas','stock','publicado_en'];

    protected function casts(): array
    {
        return [
            'publicado_en' => 'date:d-m-Y',
            'stock' => 'integer',
            'paginas' => 'integer',
        ];
    }
    public function autor(): BelongsTo
    {
        return $this->belongsTo(Autor::class);
    }

    public function genero(): BelongsTo
    {
        return $this->belongsTo(Genero::class);
    }

}
```