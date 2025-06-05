[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/keEqp6dJ)
[![Open in Codespaces](https://classroom.github.com/assets/launch-codespace-2972f46106e565e64193e422d61a12cf1da4916b45550586e14ef0a7c637dd04.svg)](https://classroom.github.com/open-in-codespaces?assignment_repo_id=19601153)
# SESION DE LABORATORIO N° 01: PRUEBAS UNITARIAS CON MSTEST

### Nombre: Cesar Nikolas Camac Melendez

## OBJETIVOS
  * Comprender el funcionamiento de las pruebas unitarias dentro de una aplicación utilizando el Framework de pruebas de Microsoft (MSTest).

## REQUERIMIENTOS
  * Conocimientos: 
    - Conocimientos básicos de Bash (powershell).
    - Conocimientos básicos de Contenedores (Docker).
  * Hardware:
    - Virtualization activada en el BIOS..
    - CPU SLAT-capable feature.
    - Al menos 4GB de RAM.
  * Software:
    - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior)
    - Docker Desktop 
    - Powershell versión 7.x
    - Net 6 o superior
    - Visual Studio Code

## CONSIDERACIONES INICIALES
  * Clonar el repositorio mediante git para tener los recursos necesaarios

## DESARROLLO
1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador 
2. Ejecutar el siguiente comando para crear una nueva solución
```
dotnet new sln -o MyMath
```
3. Acceder a la solución creada y ejecutar el siguiente comando para crear una nueva libreria de clases y adicionarla a la solución actual.
```
cd MyMath
dotnet new classlib -o Math.Lib
dotnet sln add .\Math.Lib\Math.Lib.csproj
```
4. Ejecutar el siguiente comando para crear un nuevo proyecto de pruebas y adicionarla a la solución actual
```
dotnet new mstest -o Math.Tests
dotnet sln add .\Math.Tests\Math.Tests.csproj
dotnet add .\Math.Tests\Math.Tests.csproj reference .\Math.Lib\Math.Lib.csproj
```
5. Iniciar Visual Studio Code (VS Code) abriendo el folder de la solución como proyecto. En el proyecto Math.Lib, si existe un archivo Class1.cs proceder a eliminarlo. Asimismo en el proyecto Math.Tests si existiese un archivo UnitTest1, tambièn proceder a aliminarlo.

6. En VS Code, en el proyecto Math.Tests añadir un nuevo archivo RooterTests.cs e introducir el siguiente código:
```C#
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Math.Tests
{
    [TestClass]
    public class RooterTests
    {
        [TestMethod]
        public void BasicRooterTest()
        {
            Rooter rooter = new Rooter();
            double expectedResult = 2.0;
            double input = expectedResult * expectedResult;
            double actualResult = rooter.SquareRoot(input);
            Assert.AreEqual(expectedResult, actualResult, delta: expectedResult / 100);
        }
    }
}
```
7. Abrir un terminal en VS Code (CTRL + Ñ) o vuelva al terminal anteriormente abierto, y ejecutar los comandos:
```
dotnet restore
dotnet test --collect:"XPlat Code Coverage"
```
8. El paso anterior debe producir un error por lo que sera necesario escribir el código mecesario para que el test funcione. 
9. En el proyecto Math.Lib, añadir un nuevo archivo Rooter.cs, con el siguiente contenido:
```C#
namespace Math.Lib
{
    public class Rooter
    {
        public double SquareRoot(double input)
        {
            return input / 2;
        }
    }
}
```
10. Seguidamente modificar el archivo RooterTests.cs y adicionar al inicio del mismo el siguiente contenido:
```C#
using Math.Lib;
```
11. Ejecutar nuevamente el pase 6 y ahora deberia devolver algo similar a lo siguiente:
```
Correctas! - Con error:     0, Superado:     2, Omitido:     0, Total:     2, Duración: 12 ms - Math.Tests.dll
```
12. Con la finalidad de aumentar la confienza en la aplicación, se ampliará el rango de pruebas para lo cual editar la clase de prueba RooterTests y adicionar los métodos siguientes:
```C#
        [TestMethod]
        public void RooterValueRange()
        {
            Rooter rooter = new Rooter();
            for (double expected = 1e-8; expected < 1e+8; expected *= 3.2)
                RooterOneValue(rooter, expected);
        }
        private void RooterOneValue(Rooter rooter, double expectedResult)
        {
            double input = expectedResult * expectedResult;
            double actualResult = rooter.SquareRoot(input);
            Assert.AreEqual(expectedResult, actualResult, delta: expectedResult / 1000);
        }
```
13. Ejecutar nuevamente el paso 6 para lo cual se obtendra un error similar al siguiente:
```
Con error! - Con error:     1, Superado:     2, Omitido:     0, Total:     3, Duracin: 30 ms - Math.Tests.dll
```
14. A fin de que las pruebas puedan ejecutarse correctamente, modificar la clase Rooter de la siguiente manera:
```C#
namespace Math.Lib
{
    public class Rooter
    {
        public double SquareRoot(double input)
        {
            double result = input;
            double previousResult = -input;
            while (System.Math.Abs(previousResult - result)
                > result / 1000)
            {
            previousResult = result;
            result = result - (result * result - input) / (2 * result);
            }
            return result;
        }
    }
}
```
15. Volver a ejecutar el paso 6 y verificar el resultado, debería ser similar a lo siguiente
```
Correctas! - Con error:     0, Superado:     3, Omitido:     0, Total:     3, Duracin: 14 ms - Math.Tests.dll
```
16. Adicionar un nuevo caso de prueba con excepción en la clase RooterTests:
```C#
        [TestMethod]
        public void RooterTestNegativeInputx()
        {
            Rooter rooter = new Rooter();
            try
            {
                rooter.SquareRoot(-10);
            }
            catch (System.ArgumentOutOfRangeException)
            {
                return;
            }
            Assert.Fail();
        }
```
17. Modificar la clase Rooter adicionando una nueva condición:
```C#
namespace Math.Lib
{
    public class Rooter
    {
        public double SquareRoot(double input)
        {
            if (input <= 0.0) throw new ArgumentOutOfRangeException();
            double result = input;
            double previousResult = -input;
            while (System.Math.Abs(previousResult - result)
                > result / 1000)
            {
            previousResult = result;
            result = result - (result * result - input) / (2 * result);
            }
            return result;
        }
    }
}
```
18. Al ejecutar las pruebas (paso 6) se obtendrá el siguiente resultado:
```
Correctas! - Con error:     0, Superado:     4, Omitido:     0, Total:     4, Duración: 13 ms - Math.Tests.dll
```
19. Finalmente proceder a verificar la cobertura, dentro del proyecto Primes.Tests se dede haber generado una carpeta o directorio TestResults, en el cual posiblemente exista otra subpcarpeta o subdirectorio conteniendo un archivo con nombre `coverage.cobertura.xml`, si existe ese archivo proceder a ejecutar los siguientes comandos desde la linea de comandos abierta anteriomente, de los contrario revisar el paso 8:
```
dotnet tool install -g dotnet-reportgenerator-globaltool
ReportGenerator "-reports:./*/*/*/coverage.cobertura.xml" "-targetdir:Cobertura" -reporttypes:HTML
```
20. El comando anterior primero proceda instalar una herramienta llamada ReportGenerator (https://reportgenerator.io/) la cual mediante la segunda parte del comando permitira generar un reporte en formato HTML con la cobertura obtenida de la ejecución de las pruebas. Este reporte debe localizarse dentro de una carpeta llamada Cobertura y puede acceder a el abriendo con un navegador de internet el archivo index.htm.

---
## Actividades Encargadas
1. Adicionar un nuevo escenario de prueba donde se maneje una excepción con un mensaje que diga "El valor ingresado es invalido, solo se puede ingresar números positivos".
2. Completar la documentación del Clases, atributos y métodos para luego generar una automatización (publish_docs.yml) que genere la documentación utilizando DocFx y la publique en una Github Page
3. Generar una automatización (publish_cov_report.yml) que: * Compile el proyecto y ejecute las pruebas unitarias, * Genere el reporte de cobertura, * Publique el reporte en Github Page
4. Generar una automatización (release.yml) que: * Genere el nuget con su codigo de matricula como version del componente, * Publique el nuget en Github Packages, * Genere el release correspondiente
