# Trabajo Práctico 8

## Prerequisitos
<img width="577" alt="Captura de pantalla 2024-10-18 a la(s) 18 23 11" src="https://github.com/user-attachments/assets/0f1108ab-d4ba-4973-a226-d2e3d837b0d9">
<img width="569" alt="Captura de pantalla 2024-10-18 a la(s) 18 23 28" src="https://github.com/user-attachments/assets/af6eca81-e5b7-426d-82fc-ea9071207171">
<img width="564" alt="Captura de pantalla 2024-10-18 a la(s) 18 24 04" src="https://github.com/user-attachments/assets/8b984651-efdc-4a83-910e-3034bfc4572f">
<img width="566" alt="Captura de pantalla 2024-10-18 a la(s) 18 24 12" src="https://github.com/user-attachments/assets/937c3779-e470-4bcc-b23f-16a4bb95ce29">
<img width="571" alt="Captura de pantalla 2024-10-18 a la(s) 18 24 39" src="https://github.com/user-attachments/assets/1da32907-9841-4279-b53d-4f0fd9330ee2">
<img width="565" alt="Captura de pantalla 2024-10-18 a la(s) 18 25 18" src="https://github.com/user-attachments/assets/a40bf8de-f7f8-45e9-a6da-ca1a248c3911">
<img width="960" alt="Captura de pantalla 2024-10-18 a la(s) 18 31 45" src="https://github.com/user-attachments/assets/e46c3a6e-7d17-4155-b82d-04db2e7304bf">
<img width="746" alt="Captura de pantalla 2024-10-18 a la(s) 18 32 03" src="https://github.com/user-attachments/assets/60259ebe-ce52-43d1-ac23-31478b5da353">
<img width="559" alt="image" src="https://github.com/user-attachments/assets/20456412-5f9d-48cc-995f-fd4632574d9a">

<img width="961" alt="Captura de pantalla 2024-10-18 a la(s) 18 32 22" src="https://github.com/user-attachments/assets/f1bfec9e-989f-4092-9f67-1cf92e856d3f">
<img width="778" alt="Captura de pantalla 2024-10-18 a la(s) 18 32 44" src="https://github.com/user-attachments/assets/cf1d4c57-7572-4f82-a450-2a53ce20cc32">
<img width="963" alt="Captura de pantalla 2024-10-18 a la(s) 18 33 27" src="https://github.com/user-attachments/assets/d9bc6bb1-da92-491e-bb95-5f0c4c6e9d5f">


### Paso 1
Se crea un directorio "docker" dentro del directorio raíz del proyecto que contiene 2 carpetas (api y front), y dentro de cada una de ellas se crean los correspondientes Dockerfile como se muestra a continuación.

<img width="959" alt="Captura de pantalla 2024-10-19 a la(s) 14 07 48" src="https://github.com/user-attachments/assets/3f801bc9-1b20-4cc9-a1e5-4afb32186f17">

<img width="1296" alt="Captura de pantalla 2024-10-18 a la(s) 18 40 38" src="https://github.com/user-attachments/assets/77ec3570-bf08-41b7-8eee-63bd611c97bd">

<img width="394" alt="Captura de pantalla 2024-10-19 a la(s) 13 58 55" src="https://github.com/user-attachments/assets/36dc5ba3-6698-4398-841d-2855dffe05d1">

<img width="753" alt="Captura de pantalla 2024-10-18 a la(s) 18 42 45" src="https://github.com/user-attachments/assets/8e7a8873-016b-4a99-96e4-62cda43fb781">

### Paso 2
Se crea un recurso Azure Container Registry(ACR) desde el portal de Azure siguiendo el instructivo 5.1.

<img width="846" alt="Captura de pantalla 2024-10-19 a la(s) 12 18 10" src="https://github.com/user-attachments/assets/9e07d3d2-e266-4aaf-9225-fbc3a6c20a52">

<img width="722" alt="Captura de pantalla 2024-10-19 a la(s) 12 20 16" src="https://github.com/user-attachments/assets/f026de58-a0d1-4a77-9851-5e22a3578d41">

<img width="465" alt="Captura de pantalla 2024-10-19 a la(s) 12 20 36" src="https://github.com/user-attachments/assets/5a227aa7-dec2-4f35-b34d-837035e0a640">

<img width="1512" alt="Captura de pantalla 2024-10-19 a la(s) 12 20 59" src="https://github.com/user-attachments/assets/2af7b194-c06c-4812-9ac0-9e7be5a6261f">

<img width="1512" alt="Captura de pantalla 2024-10-19 a la(s) 12 24 58" src="https://github.com/user-attachments/assets/97d7ba6e-87e7-4831-87e4-e2d77db1e6c5">

<img width="1244" alt="Captura de pantalla 2024-10-19 a la(s) 12 26 25" src="https://github.com/user-attachments/assets/d984227f-67bf-4f41-b311-d43b1574b2de">
<img width="963" alt="Captura de pantalla 2024-10-18 a la(s) 18 33 27" src="https://github.com/user-attachments/assets/f6343b01-b485-494f-962b-fb8639132f10">

### Paso 3
Luego de publicar los artefactos de compilación del backend, se agrega el siguiente código dentro del pipeline.

```
- task: PublishPipelineArtifact@1
    displayName: 'Publicar Dockerfile de Back'
    inputs:
    targetPath: '$(Build.SourcesDirectory)/docker/api/dockerfile'
    artifact: 'dockerfile-back'
```

Luego de publicar los artefactos del frontend, se agrega el siguiente código dentro del pipeline.

```
- task: PublishPipelineArtifact@1
    displayName: 'Publicar Dockerfile de Front'
    inputs:
    targetPath: '$(Build.SourcesDirectory)/docker/front/Dockerfile'
    artifact: 'dockerfile-front'
```

Estos publican los Dockerfiles anteriormente creados, lo cual permite que estén disponibles para ser utilizados en etapas posteriores.

### Paso 4
Se agrega una service connection para el manejo de recursos a Azure Portal llamada "AzureResourceManager" como se indica en instructivo 5.2.


<img width="1512" alt="Captura de pantalla 2024-10-19 a la(s) 12 29 35" src="https://github.com/user-attachments/assets/e6f94e03-c441-4bec-b926-e21ffdbf6b33">

<img width="477" alt="Captura de pantalla 2024-10-19 a la(s) 12 33 16" src="https://github.com/user-attachments/assets/68fd19a4-b6cc-4757-8111-a95b2267e34f">

<img width="485" alt="Captura de pantalla 2024-10-19 a la(s) 12 42 05" src="https://github.com/user-attachments/assets/d9742159-aab0-46af-b671-cd0a1ba0132b">

<img width="473" alt="Captura de pantalla 2024-10-19 a la(s) 12 42 32" src="https://github.com/user-attachments/assets/f1d35e67-1985-4d0b-b8be-2255a336458b">

Tenemos que modificar program.cs para que lea la variable de entorno y use esa cadena de conexion

Para que funcione, hay que habilitar el acceso administrativo de ACR desde azure CLI
<img width="747" alt="image" src="https://github.com/user-attachments/assets/60da3712-5911-4523-bb49-43ecbf0f41bc">

Ademas, debemos registrar el proveedor de recursos Microsoft.ContainerInstance, que es necesario para crear instancias de contenedores en Azure.
<img width="754" alt="image" src="https://github.com/user-attachments/assets/fec029d3-c5da-4406-9047-ee403046f5e5">


<img width="1498" alt="Captura de pantalla 2024-10-19 a la(s) 13 01 14" src="https://github.com/user-attachments/assets/b6c0faa9-ef54-4ffa-b193-46217dfd6e20">

<img width="579" alt="Captura de pantalla 2024-10-19 a la(s) 13 23 28" src="https://github.com/user-attachments/assets/2009ef72-da47-4512-bdc5-82b5d5d96c66">

### Paso 5
Se agregan las siguientes variables dentro del pipeline a las anteriormente creadas.

```
variables:
  ConnectedServiceName: 'ServiceConnectionARM'
  acrLoginServer: 'vgingsw3uccacr.azurecr.io'
  backImageName: 'employee-crud-api'
```

<img width="1512" alt="Captura de pantalla 2024-10-19 a la(s) 13 25 26" src="https://github.com/user-attachments/assets/d7759242-d570-4b62-8ab7-b767634cd2e7">

### Paso 6
Se agrega una nueva etapa al pipeline en la cual se construye y publica el contenedor backend de Docker en el recurso ACR anteriormente creado.

```
- stage: DockerBuildAndPush
  displayName: 'Construir y Subir Imágenes Docker a ACR'
  dependsOn: BuildAndTest
  jobs:
    - job: docker_build_and_push
      displayName: 'Construir y Subir Imágenes Docker a ACR'
      pool:
        vmImage: 'ubuntu-latest'
        
      steps:
        - checkout: self

        - task: DownloadPipelineArtifact@2
          displayName: 'Descargar Artefactos de Back'
          inputs:
            buildType: 'current'
            artifactName: 'dotnet-drop'
            targetPath: '$(Pipeline.Workspace)/dotnet-drop'
        
        - task: DownloadPipelineArtifact@2
          displayName: 'Descargar Dockerfile de Back'
          inputs:
            buildType: 'current'
            artifactName: 'dockerfile-back'
            targetPath: '$(Pipeline.Workspace)/dockerfile-back'

        - task: AzureCLI@2
          displayName: 'Iniciar Sesión en Azure Container Registry (ACR)'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              az acr login --name $(acrLoginServer)
    
        - task: Docker@2
          displayName: 'Construir Imagen Docker para Back'
          inputs:
            command: build
            repository: $(acrLoginServer)/$(backImageName)
            dockerfile: $(Pipeline.Workspace)/dockerfile-back/dockerfile
            buildContext: $(Pipeline.Workspace)/dotnet-drop
            tags: 'latest'

        - task: Docker@2
          displayName: 'Subir Imagen Docker de Back a ACR'
          inputs:
            command: push
            repository: $(acrLoginServer)/$(backImageName)
            tags: 'latest'
```

### Paso 7
Se ejecuta el pipeline con las modificaciones anteriormente descritas, el cual finaliza con éxito.

<img width="1257" alt="Captura de pantalla 2024-10-20 a la(s) 10 05 24" src="https://github.com/user-attachments/assets/04bb0dbf-73ba-432c-9d3d-136221260876">

Se verifica que dentro del recurso ACR anteriormente creado haya al acceder a la opción Repositorios de nuestro recurso Azure Container Registry, una imagen con el nombre definido dentro de la variable del pipeline.

<img width="1505" alt="Captura de pantalla 2024-10-20 a la(s) 10 05 37" src="https://github.com/user-attachments/assets/c2acfe82-9c24-454b-9d35-308518801afc">

### Paso 8
A la etapa anteriormente creada (), se le agrega un nuevo job que permite construir y subir la imagen de frontend de Docker al recurso ACR.

```
- job: docker_build_and_push_front
      displayName: 'Construir y Subir Imagen Docker de Front a ACR'
      pool:
        vmImage: 'ubuntu-latest'
        
      steps:
        - checkout: self

        - task: DownloadPipelineArtifact@2
          displayName: 'Descargar Artefactos de Front'
          inputs:
            buildType: 'current'
            artifactName: 'angular-drop'
            targetPath: '$(Pipeline.Workspace)/angular-drop'
        
        - task: DownloadPipelineArtifact@2
          displayName: 'Descargar Dockerfile de Back'
          inputs:
            buildType: 'current'
            artifactName: 'dockerfile-front'
            targetPath: '$(Pipeline.Workspace)/dockerfile-front'

        - task: AzureCLI@2
          displayName: 'Iniciar Sesión en Azure Container Registry (ACR)'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              az acr login --name $(acrLoginServer)
    
        - task: Docker@2
          displayName: 'Construir Imagen Docker para Back'
          inputs:
            command: build
            repository: $(acrLoginServer)/$(frontImageName)
            dockerfile: $(Pipeline.Workspace)/dockerfile-front/Dockerfile
            buildContext: $(Pipeline.Workspace)/angular-drop
            tags: 'latest'

        - task: Docker@2
          displayName: 'Subir Imagen Docker de Back a ACR'
          inputs:
            command: push
            repository: $(acrLoginServer)/$(frontImageName)
            tags: 'latest'
```
Además, se agregó una nueva variable donde se define el nombre de la imagen de frontend que será subida al ACR.

```
frontImageName: 'employee-crud-angular'
```

Se ejecuta el pipeline y se observa que el mismo finaliza de manera exitosa.

<img width="1507" alt="Captura de pantalla 2024-10-20 a la(s) 10 55 10" src="https://github.com/user-attachments/assets/ebffda5c-3daa-43c4-aa2c-0b98b889f66d">

A su vez, dentro del recurso ACR se pueden ver ambas imágenes correctamente subidas.

<img width="1509" alt="Captura de pantalla 2024-10-20 a la(s) 10 55 32" src="https://github.com/user-attachments/assets/98cecde8-3f62-4e81-b959-2615f7ad2e98">

### Paso 9
Se agregaron al pipeline las siguientes variables:

```
  acrName: 'mcingsw3uccacr'
  ResourceGroupName: 'IngSw3-UCC'
  backContainerInstanceNameQA: 'gaggio-crud-api-qa'
  backImageTag: 'latest' 
  container-cpu-api-qa: 1 #CPUS de nuestro container de QA
  container-memory-api-qa: 1.5 #RAM de nuestro container de QA
```

Desde la interfaz gráfica de Azure DevOps, se agregó una variable secreta "cnn_string_qa" que contiene la cadena de conexión a la base de datos de Azure. Imagen Paso 9a

Finalmente, se agregó una nueva etapa al pipeline que se encarga de realizar el despliegue de la imagen anteriormente creada en Azure Container Instances, dentro del entorno de QA.

```
- stage: DeployToACIQA
  displayName: 'Desplegar en Azure Container Instances (ACI) QA'
  dependsOn: DockerBuildAndPush
  jobs:
    - job: deploy_to_aci_qa
      displayName: 'Desplegar en Azure Container Instances (ACI) QA'
      pool:
        vmImage: 'ubuntu-latest'
        
      steps:

        - task: AzureCLI@2
          displayName: 'Desplegar Imagen Docker de Back en ACI QA'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              echo "Resource Group: $(ResourceGroupName)"
              echo "Container Instance Name: $(backContainerInstanceNameQA)"
              echo "ACR Login Server: $(acrLoginServer)"
              echo "Image Name: $(backImageName)"
              echo "Image Tag: $(backImageTag)"
              echo "Connection String: $(cnn_string_qa)"
          
              az container delete --resource-group $(ResourceGroupName) --name $(backContainerInstanceNameQA) --yes

              az container create --resource-group $(ResourceGroupName) \
                --name $(backContainerInstanceNameQA) \
                --image $(acrLoginServer)/$(backImageName):$(backImageTag) \
                --registry-login-server $(acrLoginServer) \
                --registry-username $(acrName) \
                --registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv) \
                --dns-name-label $(backContainerInstanceNameQA) \
                --ports 80 \
                --environment-variables ConnectionStrings__DefaultConnection="$(cnn_string_qa)" \
                --restart-policy Always \
                --cpu $(container-cpu-api-qa) \
                --memory $(container-memory-api-qa)
```

### Paso 10
Se ejecuta el pipeline con las modificaciones anteriormente mencionadas, que finaliza de manera exitosa.

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 21 49 18" src="https://github.com/user-attachments/assets/96f9b701-8887-4ef0-b8d1-57568ab6f451">

Dentro de Azure Portal, se observa que hay un nuevo recurso Azure Container Instanes con el nombre definido dentro del pipeline.

<img width="1089" alt="Captura de pantalla 2024-10-20 a la(s) 21 48 52" src="https://github.com/user-attachments/assets/85233deb-0c67-4cf8-b734-2480500d7eaf">

Se navega a la URL provista y se observa que el contenedor se encuentra correctamente conectado a la base de datos.

 <img width="694" alt="image" src="https://github.com/user-attachments/assets/47be065a-b94d-48cf-bcf0-7305257c841f">

### Paso 11
A continuación, realizaremos el despliegue del frontend del proyecto en un Azure Container Instance utilizando variables de entorno para la API URL, lo cual permitirá utilizar el mismo archivo para múltiples despliegues en diferentes entornos. Para esto, fue necesario modificar o crear los siguientes archivos.

- src/environments/environments.ts
```
export const environment = {
  production: false,
  apiUrl: (typeof window !== 'undefined' && window.env && window.env.apiUrl) 
             ? window.env.apiUrl
             : 'http://localhost:7150/api/Employee' // Valor por defecto
};
```
- src/environments/environments.prod.ts
```
export const environment = {
  production: true,
  apiUrl: (typeof window !== 'undefined' && window.env && window.env.apiUrl) 
             ? window.env.apiUrl
             : 'http://localhost:7150/api/Employee' // Valor por defecto
};
```
-src/typing.d.ts
```
interface Window {
    env: {
      apiUrl: string;
    };
  }
```
Luego de modificar el código, se crea un nuevo job en el pipeline que crea un Azure Container Instance y realiza el deploy de la imagen de Frontend.

Nuevas variables:
```
  frontContainerInstanceNameQA: 'gaggio-crud-front-qa'
  frontImageTag: 'latest' 
  container-cpu-front-qa: 1
  container-memory-front-qa: 1.5
```
Job creado:

```
    - job: deploy_front_to_aci_qa
      displayName: 'Desplegar Frontend en Azure Container Instances (ACI) QA'
      pool:
        vmImage: 'ubuntu-latest'
        
      steps:

        - task: AzureCLI@2
          displayName: 'Desplegar Imagen Docker de Front en ACI QA'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              echo "Resource Group: $(ResourceGroupName)"
              echo "Container Instance Name: $(frontContainerInstanceNameQA)"
              echo "ACR Login Server: $(acrLoginServer)"
              echo "Image Name: $(frontImageName)"
              echo "Image Tag: $(frontImageTag)"
          
              az container delete --resource-group $(ResourceGroupName) --name $(frontContainerInstanceNameQA) --yes

              az container create --resource-group $(ResourceGroupName) \
                --name $(frontContainerInstanceNameQA) \
                --image $(acrLoginServer)/$(frontImageName):$(frontImageTag) \
                --registry-login-server $(acrLoginServer) \
                --registry-username $(acrName) \
                --registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv) \
                --dns-name-label $(frontContainerInstanceNameQA) \
                --ports 80 \
                --environment-variables API_URL="$(api_url_aci_qa)" \
                --restart-policy Always \
                --cpu $(container-cpu-front-qa) \
                --memory $(container-memory-front-qa)
```

Además, se creó una nueva variable de entorno mediante la GUI de Azure llamada "api_url_aci_qa" que contiene la URL de la API desplegada en el recurso ACI anteriormente creado.

Luego de ejecutar el pipeline, dentro de Azure Portal se creó un nuevo recurso ACI.

![image](https://github.com/user-attachments/assets/a9dfe13c-a864-460e-b8bb-56ae64fd3a39)

Al navegar a la URL del contenedor, se muestra que el mismo está correctamente conectado al backend debido a que se ven los datos almacenados en la DB, además de contener el log con la URL extraída de la variable global de Azure Pipeline.

<img width="846" alt="image" src="https://github.com/user-attachments/assets/14df20f4-bc40-43b3-88a8-083877d6ce32">

### Paso 12
Para agregar la ejecución de las pruebas de integración en el entorno de QA de los Azure Container Instances, se modificaron los archivos de Cypress para permitir que los mismos reciban variables de entorno donde se definen las URL de los contenedores (o cualquier entorno en el que se deban realizar las pruebas).

```
  const homeURL = Cypress.env('homeUrl');
  const apiURL = Cypress.env('apiUrl');
```
Luego, se agregó en el pipeline un nuevo job posterior a los despliegues en ACI, de los cuales depende. En este, se ejecutan y publican las pruebas de integración anteriormente escritas.

```
    - job: RunCypressTests
      displayName: 'Run Cypress Tests'
      dependsOn: [deploy_front_to_aci_qa, deploy_api_to_aci_qa]
      condition: succeeded()
      steps:
        - script: npm install ts-node typescript --save-dev
          displayName: 'Install typescript'
          workingDirectory: '$(frontPath)'

        - script: npx cypress run --env apiUrl=$(api_url_aci_qa),homeUrl=$(front_url_aci_qa)
          workingDirectory: '$(frontPath)'
          displayName: 'Run integration tests'
          continueOnError: true
        - task: PublishTestResults@2
          inputs:
            testResultsFormat: 'JUnit'
            testResultsFiles: '*.xml'
            searchFolder: '$(frontPath)/cypress/results'
            testRunTitle: 'Cypress Integration Tests'
```

Luego de ejecutar el pipeline, en el apartado Tests se observan los resultados publicados de las pruebas de Cypress junto con los tests unitarios.

<img width="1392" alt="Captura de pantalla 2024-10-21 a la(s) 23 05 35" src="https://github.com/user-attachments/assets/4b4d7a63-7af4-4ac9-9531-a5ce74667b02">

### Paso 13
Finalmente, se agrega una nueva etapa que realiza un despliegue de la aplicación en Azure Container Instances en un ambiente de producción. La misma es similar a la anteriormente definida, pero se agrega como requisito una aprobación manual del usuario antes de comenzar el despliegue mediante el uso de environments.

Al ejecutar el pipeline, se observa que el mismo se detiene al iniciar esta etapa permitiendo aprobar o denegar el despliegue.
![image](https://github.com/user-attachments/assets/730c96c5-a378-4c4e-ac52-4b014af81396)

![image](https://github.com/user-attachments/assets/f2e8f347-d7d8-45ff-baec-5a362140586a)

Imagen Paso 13a

Una vez aprobado el despliegue y finalizada esta etapa, dentro del grupo de recursos en Azure Portal se observa que ambos ACI fueron creados exitosamente.

gaggio-crud-api-prod.eastus.azurecontainer.io/api/Employee/GetAll
Al navegar a la URL provista por el contenedor de Front, se puede ver que los datos cargan correctamente y que los mismos están siendo obtenidos del contenedor PROD.

<img width="822" alt="image" src="https://github.com/user-attachments/assets/bb7e9bae-60de-4065-be8c-d04284ce476b">
