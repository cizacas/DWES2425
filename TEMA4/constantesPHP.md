# Declaración de constantes en PHP - diferencias entre define y const

En PHP, define y const se utilizan para crear constantes, pero tienen algunas diferencias clave:

## define
define es una función que se utiliza para definir constantes en PHP.
Se puede usar tanto dentro como fuera de las clases.
Las constantes definidas con define son globales y se pueden acceder desde cualquier parte del código

```php
define("PI", 3.14159);
echo PI; // Imprimirá 3.14159

```
## const
const se utiliza para definir constantes en el ámbito de una clase (constantes de clase).
Solo se puede utilizar dentro de una clase.
Las constantes definidas con const son específicas de la clase y se pueden acceder utilizando el operador de resolución de ámbito `::`.
El valor de una constante definida con const se evalúa en tiempo de compilación y debe ser una expresión constante. No se pueden asignar valores calculados en tiempo de ejecución.

```php
class MiClase {
    const PI = 3.14159;
}

echo MiClase::PI; // Imprimirá 3.14159
```
En resumen, la principal diferencia entre `define` y `const` en PHP es que `define` se utiliza para crear constantes globales, mientras que `const` se utiliza para crear constantes de clase en el ámbito de una clase. Las constantes definidas con `const` se evalúan en tiempo de compilación y deben ser expresiones constantes, mientras que las constantes definidas con `define` se evalúan en tiempo de ejecución y pueden contener valores calculados.