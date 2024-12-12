# Instalar Xdebug en Visual Studio Code

Para utilizar el depurador (debugger) en PHP, puedes seguir estos pasos:

1. **Descargar Xdebug**:
   - Ve a [xdebug.org](https://xdebug.org/download) y descarga la versión de Xdebug que corresponde a tu versión de PHP. Puedes usar la herramienta de diagnóstico en la página para encontrar la versión correcta.

2. **Instalar Xdebug**:
   - Copia el archivo `php_xdebug.dll` descargado en la carpeta de extensiones de PHP. Normalmente, esta carpeta se encuentra en `C:\xampp\php\ext` si usas XAMPP.

3. **Configurar PHP**:
   - Abre tu archivo `php.ini` (normalmente en `C:\xampp\php\php.ini` si usas XAMPP) y añade las siguientes líneas al final del archivo:
     ```ini
     [xdebug]
     zend_extension = "C:\xampp\php\ext\php_xdebug.dll"
     xdebug.mode = debug
     xdebug.start_with_request = yes
     xdebug.client_host = 127.0.0.1
     xdebug.client_port = 9003
     ```

4. **Configurar Visual Studio Code**:
   - Instala la extensión "PHP Debug" de Felix Becker desde el marketplace de VS Code.
   - Añade una configuración de depuración en tu archivo `launch.json`:
     ```json
     {
         "version": "0.2.0",
         "configurations": [
             {
                 "name": "Listen for Xdebug",
                 "type": "php",
                 "request": "launch",
                 "port": 9003
             }
         ]
     }
     ```
    Para configurar este archivo `launch.json` si vamos a la opción de visual studio **Run and Debug**  te da la opción de crearlo.
    ![imagen configurar launch](img/launch-configuration.png)
    Se crea dentro de la carpeta .vscode que se creará en el directorio raíz de tu proyecto.
    [Más información](https://code.visualstudio.com/docs/editor/debugging)

5. **Iniciar la depuración**:
   - Coloca puntos de interrupción (breakpoints) en tu código PHP haciendo clic en el margen izquierdo de la línea donde deseas detener la ejecución.
   - Inicia la depuración seleccionando la configuración "Listen for Xdebug" y haciendo clic en el botón de depuración en la barra lateral de VS Code.

6. **Reiniciar el servidor web**:
   - Si usas XAMPP, reinicia Apache desde el panel de control de XAMPP para aplicar los cambios.
   - Si usas el que provee php también debe ser reiniciado

7. **Ejecutar tu script PHP**:
   - Accede a tu script PHP desde el navegador o la línea de comandos. Xdebug se conectará a VS Code y detendrá la ejecución en los puntos de interrupción que hayas establecido.

