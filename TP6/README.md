# TP6

## Desarrollo:
<img width="756" alt="image" src="https://github.com/user-attachments/assets/c5fe545e-2dbf-4a30-9da4-19182c026d34">

<img width="758" alt="image" src="https://github.com/user-attachments/assets/fd9ddd64-dee8-4d5a-96ef-61f4ea94e5f8">

<img width="760" alt="image" src="https://github.com/user-attachments/assets/0b16622c-5e6b-41d9-8cd1-424f65631b0f">

1.1 Creación de una BD SQL Server para nuestra App
2.2 Obtener nuestra App
A. Clonar el repo https://github.com/ingsoft3ucc/Angular_WebAPINetCore8_CRUD_Sample.git



B. Seguir las instrucciones del README.md del repo clonado prestando atención a la modificación de la cadena de conexión en el appSettings.json para que apunte a la BD creada en 1.1

Primero, iniciamos el contenedor de SQL Server.

<img width="757" alt="image" src="https://github.com/user-attachments/assets/aeaeb3ab-3f44-41fe-87c9-86d13c436e33">

<img width="988" alt="image" src="https://github.com/user-attachments/assets/c3ca72f8-e3c4-4b3c-8acc-ed3a177dad4e">

<img width="755" alt="image" src="https://github.com/user-attachments/assets/33804b19-eff7-4ca7-958a-34325be41af1">

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/ec5ab839-991a-4d3c-ad46-d2b0e1021019">

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/9aa8da61-246e-49cc-996c-72a158409a12">

<img width="752" alt="image" src="https://github.com/user-attachments/assets/ebe4b443-bc27-42e0-aaa3-30ed66ee6b9b">

No funciono, probe de la otra forma...

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/0185c667-fffd-49aa-918a-bc1aafb6ad26">

<img width="1299" alt="image" src="https://github.com/user-attachments/assets/5526905d-eb99-4847-b3e2-7d3874e1b2e1">

<img width="1097" alt="image" src="https://github.com/user-attachments/assets/bc186834-1b18-4588-b639-3acbbb48b89e">

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/04bb7474-ebd5-4012-82b8-7a28349ae66f">

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/fefc0f9b-9647-4f76-bca2-17d9a51eb56e">

Ahora corremos el script dado por el profe.
<img width="1510" alt="image" src="https://github.com/user-attachments/assets/ba97341b-95a7-4cb3-81ab-5384a4d538a9">
<img width="1512" alt="image" src="https://github.com/user-attachments/assets/c0787f0c-7aba-4fd7-9902-483edf697fff">

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/342c6161-ce34-4809-a233-b57d0e800c20">

<img width="1512" alt="image" src="https://github.com/user-attachments/assets/f58d1c26-8bc7-46ef-b58b-094779d6dfae">


Una vez hecho esto, modificamos la cadena de conexion en el appSettings.json

<img width="1510" alt="image" src="https://github.com/user-attachments/assets/7153889e-e73d-4885-ab3d-5300f28f0390">



C. Navegar a http://localhost:7150/swagger/index.html y probar uno de los controladores para verificar el correcto funcionamiento de la API.
<img width="1190" alt="image" src="https://github.com/user-attachments/assets/ba651bdf-5776-4574-ae79-474f9a46ab34">



D. Navegar a http://localhost:4200 y verificar el correcto funcionamiento de nuestro front-end Angular

<img width="934" alt="image" src="https://github.com/user-attachments/assets/42823712-ebe9-4b65-a089-b83066783970">



Agregamos un nuevo empleado para checkear que funcione correctamente

![image](https://github.com/user-attachments/assets/bf9d2eb8-ebf1-4809-b23b-7c74242af943)


E. Una vez verificado el correcto funcionamiento de la Aplicación procederemos a crear un proyecto de pruebas unitarias para nuestra API.

<img width="1480" alt="image" src="https://github.com/user-attachments/assets/9e954826-1bbe-4354-b34f-1b3856f5e900">



1.3 Crear Pruebas Unitarias para nuestra API
A. En el directorio raiz de nuestro repo crear un nuevo proyecto de pruebas unitarias para nuestra API

dotnet new xunit -n EmployeeCrudApi.Tests


B. Instalar dependencias necesarias
![Uploading image.png…]()



Y lo mismo con las demas dependencias.

C. Editar archivo UnitTest1.cs reemplazando su contenido por:

using EmployeeCrudApi.Controllers;
using EmployeeCrudApi.Data;
using EmployeeCrudApi.Models;
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Xunit;

namespace EmployeeCrudApi.Tests
{
    public class EmployeeControllerTests
    {
        private ApplicationDbContext GetInMemoryDbContext()
        {
            var options = new DbContextOptionsBuilder<ApplicationDbContext>()
                .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString()) // Crear una nueva base de datos en memoria para cada prueba
                .Options;

            return new ApplicationDbContext(options);
        }

        [Fact]
        public async Task GetAll_ReturnsListOfEmployees()
        {
            // Arrange
            var context = GetInMemoryDbContext();
            context.Employees.AddRange(
                new Employee { Id = 1, Name = "John Doe" },
                new Employee { Id = 2, Name = "Jane Doe" }
            );
            context.SaveChanges();

            var controller = new EmployeeController(context);

            // Act
            var result = await controller.GetAll();

            // Assert
            Assert.Equal(2, result.Count);
            Assert.Equal("John Doe", result[0].Name);
            Assert.Equal("Jane Doe", result[1].Name);
        }

        [Fact]
        public async Task GetById_ReturnsEmployeeById()
        {
            // Arrange
            var context = GetInMemoryDbContext();
            context.Employees.Add(new Employee { Id = 1, Name = "John Doe" });
            context.SaveChanges();

            var controller = new EmployeeController(context);

            // Act
            var result = await controller.GetById(1);

            // Assert
            Assert.NotNull(result);
            Assert.Equal(1, result.Id);
            Assert.Equal("John Doe", result.Name);
        }

        [Fact]
        public async Task Create_AddsEmployee()
        {
            // Arrange
            var context = GetInMemoryDbContext();
            var controller = new EmployeeController(context);

            var newEmployee = new Employee { Id = 3, Name = "New Employee" };

            // Act
            await controller.Create(newEmployee);

            // Assert
            var employee = await context.Employees.FindAsync(3);
            Assert.NotNull(employee);
            Assert.Equal("New Employee", employee.Name);
        }

        [Fact]
        public async Task Update_UpdatesEmployee()
        {
            // Arrange
            var context = GetInMemoryDbContext();
            var existingEmployee = new Employee { Id = 1, Name = "Old Name" };
            context.Employees.Add(existingEmployee);
            context.SaveChanges();

            var controller = new EmployeeController(context);

            var updatedEmployee = new Employee { Id = 1, Name = "Updated Name" };

            // Act
            await controller.Update(updatedEmployee);

            // Assert
            var employee = await context.Employees.FindAsync(1);
            Assert.NotNull(employee);
            Assert.Equal("Updated Name", employee.Name);
        }

        [Fact]
        public async Task Delete_RemovesEmployee()
        {
            // Arrange
            var context = GetInMemoryDbContext();
            var employeeToDelete = new Employee { Id = 1, Name = "John Doe" };
            context.Employees.Add(employeeToDelete);
            context.SaveChanges();

            var controller = new EmployeeController(context);

            // Act
            await controller.Delete(1);

            // Assert
            var employee = await context.Employees.FindAsync(1);
            Assert.Null(employee); // Verifica que el empleado fue eliminado
        }
    }
}


D. Renombrar archivo UnitTest1.cs por EmployeeControllerUnitTests.cs

mv UnitTest1.cs EmployeeControllerUnitTests.cs 
E. Editar el archivo EmployeeCrudApi.Tests/EmployeeCrudApi.Tests.csproj para agregar una referencia a nuestro proyecto de EmployeeCrudApi reemplazando su contenido por

<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>

    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="coverlet.collector" Version="6.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="8.0.8" />
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="17.8.0" />
    <PackageReference Include="Moq" Version="4.20.71" />
    <PackageReference Include="xunit" Version="2.9.0" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.5.3" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="../EmployeeCrudApi/EmployeeCrudApi/EmployeeCrudApi.csproj" />
  </ItemGroup>

  <ItemGroup>
    <Using Include="Xunit" />
  </ItemGroup>

</Project>


F. Ejecutar los siguientes comandos para ejecutar nuestras pruebas

dotnet build
dotnet test
G. Verificar que se hayan ejecutado correctamente las pruebas



I. Modificar la cadena de conexión en el archivo appsettings.json para que use un usuario o password incorrecto y recompilar el proyecto EmployeeCrudApi

dotnet build
dotnet run --urls "http://localhost:7150"


J. Verificar que nuestro proyecto ya no tiene acceso a la BD navegando a http://localhost:7150/swagger/index.html y probando uno de los controladores:

Si probamos la API vemos que nos devuelve un HTPP CODE 500, es decir, no funciona por problemas internos del servidor.



K. En la carpeta de nuestro proyecto EmployeeCrudApi.Tests volver a correr las pruebas

dotnet build
dotnet test
L. Verificar que se hayan ejecutado correctamente las pruebas inclusive sin tener acceso a la BD, lo que confirma que es efectivamente un conjunto de pruebas unitarias que no requieren de una dependencia externa para funcionar.



M. Modificar la cadena de conexión en el archivo appsettings.json para que use el usuario y password correcto y recompilar el proyecto EmployeeCrudApi

dotnet build
dotnet run --urls "http://localhost:7150"


N. Verificar que nuestro proyecto vuelve a tener acceso a la BD navegando a http://localhost:7150/swagger/index.html y probando uno de los controladores:



1.4 Creamos pruebas unitarias para nuestro front de Angular:
Intro. Para las pruebas unitarias de nuestro front en Angular utilizaremos Jasmine y Karma, herramientas ampliamente utilizadas para pruebas unitarias en aplicaciones web, especialmente en proyectos de Angular.

Jasmine: Es un framework de pruebas unitarias para JavaScript. Es popular porque no requiere dependencias externas y es fácil de usar. Jasmine se utiliza para escribir pruebas de manera estructurada y ofrece un conjunto de funciones para facilitar la verificación del comportamiento del código.

Características principales:
Sintaxis descriptiva: Jasmine permite escribir pruebas en un estilo de "BDD" (Desarrollo Orientado a Comportamiento). Usa palabras clave como describe, it, expect para definir pruebas de forma muy legible.
Sin dependencias externas: A diferencia de otros frameworks de prueba, Jasmine no necesita ninguna biblioteca externa para funcionar.
Espías (Spies): Jasmine permite observar llamadas a funciones, rastrearlas y verificar si fueron llamadas correctamente.
Karma: Es un test runner (ejecutor de pruebas) que facilita la ejecución de pruebas en varios navegadores. Está diseñado para trabajar con frameworks de pruebas como Jasmine, Mocha, entre otros. Karma se integra perfectamente con proyectos de Angular y es parte del Angular CLI.

Características principales:
Soporte de múltiples navegadores: Karma permite ejecutar pruebas en diferentes navegadores (Chrome, Firefox, Safari, etc.) de forma automática.
Ejecución en modo de observación: Karma puede detectar cambios en los archivos y ejecutar las pruebas automáticamente en cuanto se modifican.
Generación de informes: Karma proporciona informes detallados sobre el resultado de las pruebas.
Integración con CI/CD: Se puede integrar con sistemas de integración continua como Azure Devops, Jenkins, Travis CI o CircleCI para ejecutar las pruebas automáticamente en un entorno controlado.
Flujo de trabajo con Karma:
Karma inicia un servidor.
Abre un navegador (o varios) y ejecuta las pruebas en cada uno de ellos.
Karma se queda a la espera de cambios en el código, ejecutando nuevamente las pruebas si detecta modificaciones.
A. Nos posicionamos en nuestro proyecto de front, en el directorio EmployeeCrudAngular/src/app

B. Editamos el archivo app.component.spec.ts

import { TestBed } from '@angular/core/testing';
import { AppComponent } from './app.component'; // Ajusta la ruta si es necesario

describe('AppComponent', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [AppComponent], // Usa imports en lugar de declarations
    }).compileComponents();
  });

  it('should render title', () => {
    const fixture = TestBed.createComponent(AppComponent);
    fixture.detectChanges();
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('h1')?.textContent).toContain('EmployeeCrudAngular');
  });

});


C. Creamos el archivo employee.service.spec.ts reemplazando su contenido por:

import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { EmployeeService } from './employee.service';
import { Employee } from './employee.model';
import { DatePipe } from '@angular/common';

describe('EmployeeService', () => {
  let service: EmployeeService;
  let httpMock: HttpTestingController;
  let datePipe: DatePipe;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        EmployeeService,
        DatePipe
      ]
    });

    service = TestBed.inject(EmployeeService);
    httpMock = TestBed.inject(HttpTestingController);
    datePipe = TestBed.inject(DatePipe);
  });

  afterEach(() => {
    httpMock.verify();
  });



  it('should retrieve all employees', () => {
    const today = new Date();
    const expectedDateTime = datePipe.transform(today, 'dd/MM/yyyy HH:mm:ss', undefined) ?? '';  // Consistente con el servicio

    const dummyEmployees: Employee[] = [
      new Employee(1, 'John Doe', expectedDateTime),
      new Employee(2, 'Jane Smith', expectedDateTime)
    ];

    service.getAllEmployee().subscribe(employees => {
      expect(employees.length).toBe(2);
      employees.forEach((employee, index) => {
        // Agrega depuración aquí
        console.log('Employee createdDate:', datePipe.transform(employee.createdDate, 'dd/MM/yyyy HH:mm:ss', undefined)?? '');  // Imprimir el valor generado por el servicio
        console.log('Dummy employee createdDate:', datePipe.transform(dummyEmployees[index].createdDate, 'MM/dd/yyyy HH:mm:ss', undefined)?? '');   // Imprimir el valor esperado

        expect(datePipe.transform(employee.createdDate, 'dd/MM/yyyy HH:mm:ss', undefined)?? '').toEqual(datePipe.transform(dummyEmployees[index].createdDate, 'MM/dd/yyyy HH:mm:ss', undefined)?? '');  // Compara la fecha completa
      });
    });

    const req = httpMock.expectOne(`${service.apiUrlEmployee}/getall`);
    expect(req.request.method).toBe('GET');
    req.flush(dummyEmployees);
  });


});


D. Editamos el archivo employee.component.spec.ts ubicado en la carpeta employee reemplazando su contenido por:

import { TestBed } from '@angular/core/testing';
import { EmployeeComponent } from './employee.component';
import { HttpClientTestingModule } from '@angular/common/http/testing';
import { DatePipe } from '@angular/common';

describe('EmployeeComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [EmployeeComponent, HttpClientTestingModule],
      providers: [DatePipe] // Añade DatePipe a los proveedores
    });
  });

  it('should create', () => {
    const fixture = TestBed.createComponent(EmployeeComponent);
    const component = fixture.componentInstance;
    expect(component).toBeTruthy();
  });
});


E. Editamos el archivo addemployee.component.spec.ts ubicado en la carpeta addemployee reemplazando su contenido por:

import { TestBed } from '@angular/core/testing';
import { EmployeeComponent } from './employee.component';
import { HttpClientTestingModule } from '@angular/common/http/testing';
import { DatePipe } from '@angular/common';

describe('EmployeeComponent', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [EmployeeComponent, HttpClientTestingModule],
      providers: [DatePipe] // Añade DatePipe a los proveedores
    });
  });

  it('should create', () => {
    const fixture = TestBed.createComponent(EmployeeComponent);
    const component = fixture.componentInstance;
    expect(component).toBeTruthy();
  });
});


F. En el directorio raiz de nuestro proyecto EmployeeCrudAngular ejecutamos el comando

ng test
En proyectos de Angular, Jasmine se usa para escribir las pruebas, y Karma se encarga de ejecutarlas. Cuando ejecutamos el comando ng test, Karma se inicia, carga las pruebas escritas en Jasmine, y las ejecuta en un navegador.



G. Vemos que se abre una ventana de Karma con Jasmine en la que nos indica que los tests se ejecutaron correctamente



H. Vemos que los tests se ejecutaron correctamente:



I. Verificamos que no esté corriendo nuestra API navegando a http://localhost:7150/swagger/index.html y recibiendo esta salida:



J. Los puntos G y H nos indican que se han ejecutado correctamente las pruebas inclusive sin tener acceso a la API, lo que confirma que es efectivamente un conjunto de pruebas unitarias que no requieres de una dependencia externa para funcionar.

1.5 Agregamos generación de reporte XML de nuestras pruebas de front.
Para cuando integremos nuestras pruebas en un pipeline de Build, vamos a necesitar el resultado devuelto por nuestras pruebas para reportarlas junto a las pruebas de back que se reportan automaticamente.

A. Instalamos dependencia karma-junit-reporter

npm install karma-junit-reporter --save-dev


B. En el directorio raiz de nuestro proyecto (al mismo nivel que el archivo angular.json) creamos un archivo karma.conf.js con el siguiente contenido

module.exports = function (config) {
  config.set({
    frameworks: ['jasmine', '@angular-devkit/build-angular'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-junit-reporter'),
      require('@angular-devkit/build-angular/plugins/karma')
    ],
    reporters: ['progress', 'junit'],
    junitReporter: {
      outputDir: 'test-results',
      outputFile: 'test-results.xml',
      useBrowserName: false
    },
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: true,
    browsers: ['ChromeHeadless'],
    singleRun: true,
    restartOnFileChange: true
  });
};


C. Ejecutamos nuestros test de la siguiente manera:

ng test --karma-config=karma.conf.js --watch=false --browsers ChromeHeadless


D. Verificamos que se creo un archivo test-result.xml en el directorio test-results que está al mismo nivel que el directorio src

 

1.6 Modificamos el código de nuestra API y creamos nuevas pruebas unitarias:
A. Realizar al menos 5 de las siguientes modificaciones sugeridas al código de la API:

Al agregar y al editar un empleado, controlar que el nombre del empleado no esté repetido.
La longitud máxima del nombre y apellido del empleado debe ser de 100 caracteres.
Almacenar el nombre en la BD siempre con la primera letra de los nombres en Mayuscula y todo el apellido en Mayusculas. Ejemplo, si recibo juan carlos chamizo, se debe almacenar como Juan Carlos CHAMIZO.
Asegurar que el nombre del empleado no contenga caracteres especiales o números, a menos que sea necesario (por ejemplo, caracteres especiales en apellidos como "O'Connor" o "García").
Validar que el nombre tenga un número mínimo de caracteres, por ejemplo, al menos dos caracteres para evitar entradas inválidas como "A".
Verificar que el nombre no contenga números, ya que no es común en los nombres de empleados.
Asegurar que cada parte del nombre (separada por espacios) tenga al menos un carácter o más, por ejemplo, para evitar "A B".
Verificar que no haya palabras vacías o que el nombre no esté compuesto solo de espacios.
Prohibir el uso de nombres triviales o genéricos como "Empleado", "N/A", "Nombre", etc.
Evitar que se ingresen caracteres repetidos de forma excesiva, como "Juuuuaannnn".
Implementar un filtro para evitar el uso de palabras inapropiadas, ofensivas o que puedan violar políticas internas. En todos los casos donde no se cumplan las condiciones, la API debe devolver un error de HTTP 400 Bad Request y un Json indicando el error, por ejemplo:
{
  "status": 400,
  "error": "Bad Request",
  "message": "El nombre del empleado ya existe."
}
Elegi hacer los siguientes cambios en la API.

Al agregar y al editar un empleado, controlar que el nombre del empleado no esté repetido.
La longitud máxima del nombre y apellido del empleado debe ser de 100 caracteres.
Almacenar el nombre en la BD siempre con la primera letra de los nombres en Mayuscula y todo el apellido en Mayusculas. Ejemplo, si recibo juan carlos chamizo, se debe almacenar como Juan Carlos CHAMIZO.
Asegurar que el nombre del empleado no contenga caracteres especiales o números (por ejemplo, caracteres especiales en apellidos como "O'Connor" o "García").
Verificar que el nombre no contenga números, ya que no es común en los nombres de empleados.
Validar que el nombre tenga un número mínimo de caracteres, por ejemplo, al menos dos caracteres para evitar entradas inválidas como "A".
En la siguiente imagen, se pueden ver los cambios en la funcion "Create", donde se cumplen las condiciones de arriba. Hay mas cambios en las demas funciones los cuales los puede ver en el archivo EmployeeController.cs.



B. Crear las pruebas unitarias necesarias para validar las modificaciones realizadas en el código

Cree las pruebas unitarias para validar cada modificacion. En la siguiente imagen, se pueden ver las pruebas unitarias para una validacion y para la correcta creacion de un empleado, el resto de las pruebas las puede ver en el archivo EmployeeControllerUnitTests.cs.



Si corremos las pruebas vemos:



1.7 Modificamos el código de nuestro Front y creamos nuevas pruebas unitarias:
A. Realizar en el código del front las mismas modificaciones hechas a la API.

En la siguiente imagen, se pueden ver algunas de las modificaciones realizadas, el resto las puede ver en el archivo addemployee.component.ts.



B. Las validaciones deben ser realizadas en el front sin llegar a la API, y deben ser mostradas en un toast como por ejemplo https://stackblitz.com/edit/angular12-toastr?file=src%2Fapp%2Fapp.component.ts o https://stackblitz.com/edit/angular-error-toast?file=src%2Fapp%2Fcore%2Frxjsops.ts

Vemos que se uso toastr.

     

C. Crear las pruebas unitarias necesarias en el front para validar las modificaciones realizadas en el código del front.

La siguiente imagen muestra algunas de las pruebas unitarias realizadas para validar las modificaicones en el codigo del front, el resto lo puede ver en el archivo addemployee.component.spec.ts.



Si corremos los test con el comando 'ng test', vemos lo siguiente:



Vemos que se corrieron las pruebas y pasaron todas. Si ahora lo corremos con el comando 'ng test --karma-config=karma.conf.js --watch=false --browsers ChromeHeadless' vemos:

 
