### Información de la libreria FakerPHP/Faker
**Instalación**

Faker requiere PHP = 7.4.
```shell
composer require fakerphp/faker
```
**Uso básico**

Autocarga
Faker apoya a ambos PSR-0como PSR-4autocargas.
```shell

// when installed via composer
require_once 'vendor/autoload.php';
```
**Crear datos falsos**

Uso Faker\Factory::create()crear e inicializar un generador de falsificador, que puede generar datos mediante métodos de llamada con el nombre del tipo de datos que desea.
```php
require_once 'vendor/autoload.php';

// use the factory to create a Faker\Generator instance
$faker = Faker\Factory::create();
// generate data by calling methods
echo $faker->name();
// 'Vince Sporer'
echo $faker->email();
// 'walter.sophia@hotmail.com'
echo $faker->text();
// 'Numquam ut mollitia at consequuntur inventore dolorem.'

```
Cada llamada a `$faker->name()`rinde un resultado (aleatorio) diferente. 
Esto se debe a que Faker usa `__call()`magia, y delanteros `Faker\Generator->$method()`llamadas a `Faker\Generator->format($method, $attributes)`.

```php
for ($i = 0; $i < 3; $i++) {
    echo $faker->name() . "\n";
}

// 'Cyrus Boyle'
// 'Alena Cummerata'
// 'Orlo Bergstrom'
```

**Modificadores**

Faker proporciona tres proveedores especiales, `unique()`, `optional()`, y `valid()`, para ser llamado antes de cualquier proveedor.

```php
// unique() forces providers to return unique values
$values = [];
for ($i = 0; $i < 10; $i++) {
    // get a random digit, but always a new one, to avoid duplicates
    $values []= $faker->unique()->randomDigit();
}
print_r($values); // [4, 1, 8, 5, 0, 2, 6, 9, 7, 3]

// providers with a limited range will throw an exception when no new unique value can be generated
$values = [];
try {
    for ($i = 0; $i < 10; $i++) {
        $values []= $faker->unique()->randomDigitNotNull();
    }
} catch (\OverflowException $e) {
    echo "There are only 9 unique digits not null, Faker can't generate 10 of them!";
}

// you can reset the unique modifier for all providers by passing true as first argument
$faker->unique($reset = true)->randomDigitNotNull(); // will not throw OverflowException since unique() was reset
// tip: unique() keeps one array of values per provider

// optional() sometimes bypasses the provider to return a default value instead (which defaults to NULL)
$values = [];
for ($i = 0; $i < 10; $i++) {
    // get a random digit, but also null sometimes
    $values []= $faker->optional()->randomDigit();
}
print_r($values); // [1, 4, null, 9, 5, null, null, 4, 6, null]

// optional() accepts a weight argument to specify the probability of receiving the default value.
// 0 will always return the default value; 1.0 will always return the provider. Default weight is 0.5 (50% chance).
// Please note that the weight can be provided as float (0 / 1.0) or int (0 / 100)

// As float
$faker->optional($weight = 0.1)->randomDigit(); // 90% chance of NULL
$faker->optional($weight = 0.9)->randomDigit(); // 10% chance of NULL

// As int
$faker->optional($weight = 10)->randomDigit; // 90% chance of NULL
$faker->optional($weight = 100)->randomDigit; // 0% chance of NULL

// optional() accepts a default argument to specify the default value to return.
// Defaults to NULL.
$faker->optional($weight = 0.5, $default = false)->randomDigit(); // 50% chance of FALSE
$faker->optional($weight = 0.9, $default = 'abc')->word(); // 10% chance of 'abc'

// valid() only accepts valid values according to the passed validator functions
$values = [];
$evenValidator = function($digit) {
    return $digit % 2 === 0;
};
for ($i = 0; $i < 10; $i++) {
    $values []= $faker->valid($evenValidator)->randomDigit();
}
print_r($values); // [0, 4, 8, 4, 2, 6, 0, 8, 8, 6]

// just like unique(), valid() throws an overflow exception when it can't generate a valid value
$values = [];
try {
    $faker->valid($evenValidator)->randomElement([1, 3, 5, 7, 9]);
} catch (\OverflowException $e) {
    echo "Can't pick an even number in that set!";
}
```
Si desea utilizar un modificador con un valor no generado por Faker, utilice el passthrough()método. passthrough()simplemente devuelve el valor que se le dio.

```php
$faker->optional()->passthrough(mt_rand(5, 15));

```
**Localización**

Faker\Factory puede tomar un lugar como argumento, para devolver los datos localizados. Si no se encuentra ningún proveedor localizado, el La fábrica vuelve a la locale por defecto (en US).

```php
// create a French faker
$faker = Faker\Factory::create('fr_FR');
for ($i = 0; $i < 3; $i++) {
    echo $faker->name() . "\n";
}

// Luce du Coulon
// Auguste Dupont
// Roger Le Voisin
```

**Formadores específicos del lenguaje es_ES**

```shell
Faker\Provider\es_ES\Person-

// Generates a Documento Nacional de Identidad (DNI) number
echo $faker->dni(); // '77446565E'

// Generates a random valid licence code
echo $faker->licenceCode(); // B

Faker\Provider\es_ES\Payment-

echo $faker->bankAccountNumber(); // "ES5285748762396535068585"

// Generates a Código de identificación Fiscal (CIF) number
echo $faker->vat(); // "A35864370"

Faker\Provider\es_ES\PhoneNumber-

// Generates a special rate toll free phone number
echo $faker->tollFreeNumber(); // 900 123 456

// Generates a mobile phone number
echo $faker->mobileNumber(); // +34 612 12 24
```
