
# 1- Desarrollo:
### 1.1 Modificar nuestro pipeline para incluir el deploy en QA y PROD de Imagenes Docker en Servicio Azure App Services con Soporte para Contenedores
Desarrollo del punto 1.1:

### 1.1.1 - Agregar a nuestro pipeline una nueva etapa que dependa de nuestra etapa de Construcción y Pruebas y de la etapa de Construcción de Imagenes Docker y subida a ACR realizada en el TP08
Agregar tareas para crear un recurso Azure Container Instances que levante un contenedor con nuestra imagen de back utilizando un AppServicePlan en Linux

```text
  #---------------------------------------
  ### STAGE DEPLOY TO AZURE APP SERVICE QA
  #---------------------------------------
  - stage: DeployImagesToAppServiceQA
    displayName: 'Desplegar Imagenes en Azure App Service (QA)'
    dependsOn: 
    - BuildAndTestBackAndFront
    - DockerBuildAndPush
    condition: succeeded()
    jobs:
      - job: DeployImagesToAppServiceQA
        displayName: 'Desplegar Imagenes de API y Front en Azure App Service (QA)'
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          #------------------------------------------------------
          # DEPLOY DOCKER API IMAGE TO AZURE APP SERVICE (QA)
          #------------------------------------------------------
          - task: AzureCLI@2
            displayName: 'Verificar y crear el recurso Azure App Service para API (QA) si no existe'
            inputs:
              azureSubscription: '$(ConnectedServiceName)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Verificar si el App Service para la API ya existe
                if ! az webapp list --query "[?name=='$(WebAppApiNameContainersQA)' && resourceGroup=='$(ResourceGroupName)'] | length(@)" -o tsv | grep -q '^1$'; then
                  echo "El App Service para API QA no existe. Creando..."
                  # Crear el App Service sin especificar la imagen del contenedor
                  az webapp create --resource-group $(ResourceGroupName) --plan $(AppServicePlanLinux) --name $(WebAppApiNameContainersQA) --deployment-container-image-name "nginx"  # Especifica una imagen temporal para permitir la creación
                else
                  echo "El App Service para API QA ya existe. Actualizando la imagen..."
                fi
  
                # Configurar el App Service para usar Azure Container Registry (ACR)
                az webapp config container set --name $(WebAppApiNameContainersQA) --resource-group $(ResourceGroupName) \
                  --container-image-name $(acrLoginServer)/$(backImageName):$(backImageTag) \
                  --container-registry-url https://$(acrLoginServer) \
                  --container-registry-user $(acrName) \
                  --container-registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv)
                # Establecer variables de entorno
                az webapp config appsettings set --name $(WebAppApiNameContainersQA) --resource-group $(ResourceGroupName) \
                  --settings ConnectionStrings__DefaultConnection="$(cnn-string-qa)" \

Primero, creamos un plan de linux en Azure.
<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 08 49" src="https://github.com/user-attachments/assets/dd35f6f3-0bcb-4314-83c0-00bcf3d24fd8">

 <img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 09 04" src="https://github.com/user-attachments/assets/4a52e477-ec71-4e6e-b422-303f9083a601">

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 10 04" src="https://github.com/user-attachments/assets/ce0633c3-6f53-4117-9f31-5631f98e5cf9">

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 10 33" src="https://github.com/user-attachments/assets/90cdcd49-db4d-465a-802b-f487357e0398">

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 10 48" src="https://github.com/user-attachments/assets/96ee7242-46d2-4cc4-a3fb-f754f36b541c">


Ahora agregamos al pipeline las tareas para hacer el deploy a App Service para contenedores.



Para esto, agregamos las siguientes variables.



Ahora corremos el pipeline:



Y vemos que se ejecuto correctamente.





### 1.2 Desafíos:
Para realizar los desafios use un pipeline con templates que se llama tp-09-desafio.

##### 1.2.1 Agregar tareas para generar Front en Azure App Service con Soporte para Contenedores
Agregamos las siguientes tareas:



Corremos el pipeline:



Y vemos que ahora se creo el container de front.

 

##### 1.2.2 Agregar variables necesarias para el funcionamiento de la nueva etapa considerando que debe haber 2 entornos QA y PROD para Back y Front.
Agregamos la variable de coneccion para prod.



Agregamos las API URLs



##### 1.2.3 Agregar tareas para correr pruebas de integración en el entorno de QA de Back y Front creado en Azure App Services con Soporte para Contenedores.
Agregamos las siguientes tareas para correr tests de integracion:



Corremos el pipeline:

 

Las pruebas se ejecutaron correctamente.

##### 1.2.4 Agregar etapa que dependa de la etapa de Deploy en QA que genere un entorno de PROD.
Agregamos el siguiente stage donde se hace el deploy de front y de back en PROD con aprobacion manual.

 

Corremos el pipeline y vemos que cuando pasa la etapa de QA nos pide aprobacion.
<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 23 17 21" src="https://github.com/user-attachments/assets/93d01208-930a-4b96-bce4-c0fd6d587721">

  

Vemos que funciona correctamente.



##### 1.2.5 Entregar un pipeline que incluya:
A) Etapa Construcción y Pruebas Unitarias y Code Coverage Back y Front
B) Construcción de Imágenes Docker y subida a ACR
C) Deploy Back y Front en QA con pruebas de integración para Azure Web Apps
D) Deploy Back y Front en QA con pruebas de integración para ACI
E) Deploy Back y Front en QA con pruebas de integración para Azure Web Apps con Soporte para contenedores
F) Aprobación manual de QA para los puntos C,D,E
G) Deploy Back y Front en PROD para Azure Web Apps
H) Deploy Back y Front en PROD para ACI
I) Deploy Back y Front en PROD para Azure Web Apps con Soporte para contenedores
Para este desafio, cree los siguientes templates.



Y los agregue al pipeline. Los templates para deployar a WebApps son las tareas usadas en el tp-07, y los templates de deploy a container instances son las tareas usadas en el tp-08.



Corremos el pipeline. Los deploy a QA funcionan y vemos que se ejecutaron todos los test y code coverage.

  

Vemos que pide aprobacion cuando llega a PROD.

 

Ahora corroboramos que funcionen los deploy de PROD.

Web Apps:

 

ACI:

 

Web Apps con Soporte para contenedores:

 

Una vez deployados, termina el pipeline.



Y vemos que se crearon todos los recursos en Azure.

