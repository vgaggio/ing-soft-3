## 1- Desarrollo:
### Paso 1

Se añade una nueva etapa al pipeline, la cual depende tanto de la fase de construcción y pruebas como de la fase de construcción y carga de las imágenes Docker en ACR. En esta etapa, se inicia un contenedor utilizando la imagen de la API y se despliega en un Azure App Service, para lo cual es necesario contar con un App Service Plan basado en Linux.



 ```
    #---------------------------------------
  ### STAGE DEPLOY TO AZURE APP SERVICE QA
  #---------------------------------------
- stage: DeployImagesToAppServiceQA
  displayName: 'Desplegar Imagenes en Azure App Service (QA)'
  dependsOn: 
    - BuildAndTest
    - DockerBuildAndPush
  condition: succeeded()
  jobs:
      - job: DeployAPIImageToAppServiceQA
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
```
Primero, creamos un plan de linux en Azure.
<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 08 49" src="https://github.com/user-attachments/assets/dd35f6f3-0bcb-4314-83c0-00bcf3d24fd8">

 <img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 09 04" src="https://github.com/user-attachments/assets/4a52e477-ec71-4e6e-b422-303f9083a601">

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 10 04" src="https://github.com/user-attachments/assets/ce0633c3-6f53-4117-9f31-5631f98e5cf9">

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 10 33" src="https://github.com/user-attachments/assets/90cdcd49-db4d-465a-802b-f487357e0398">

<img width="1512" alt="Captura de pantalla 2024-10-20 a la(s) 22 10 48" src="https://github.com/user-attachments/assets/96ee7242-46d2-4cc4-a3fb-f754f36b541c">

Al ejecutar este job dentro del pipeline, se crea en Azure Portal un nuevo recurso.

<img width="1505" alt="image" src="https://github.com/user-attachments/assets/ccb8e5a9-e844-475f-ae28-c12bf04e5f32">

### Paso 2
Se agrega un nuevo job a la etapa anteriormente creada que permite realizar el despliegue de un contenedor del frontend del proyecto en un Azure App Service.
 ```
# -------------------------------------------------------------------------------
# |               DEPLOY FRONTEND TO AZURE APP SERVICE                           |
# -------------------------------------------------------------------------------
      - job: DeployFrontImageToAppServiceQA
        displayName: 'Deploy Front to Azure App Service (QA)'
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          - task: AzureCLI@2
            displayName: 'Create Front Azure App Service resource and configure image'
            inputs:
              azureSubscription: '$(ConnectedServiceName)'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Check if Front App Service already exists
                if ! az webapp list --query "[?name=='$(frontAppServiceQA)' && resourceGroup=='$(ResourceGroupName)'] | length(@)" -o tsv | grep -q '^1$'; then
                  echo "Front App Service doesn't exist. Creating new..."
                  # Create App Service without specifying container image
                  az webapp create --resource-group $(ResourceGroupName) --plan $(AppServicePlanLinux) --name $(frontAppServiceQA) --deployment-container-image-name "nginx"  # Especifica una imagen temporal para permitir la creación
                else
                  echo "Front App Service already exists. Updating image..."
                fi

                # Configure App Service to use Azure Container Registry (ACR)
                az webapp config container set --name $(frontAppServiceQA) --resource-group $(ResourceGroupName) \
                  --container-image-name $(acrLoginServer)/$(frontImageName):$(frontImageTag) \
                  --container-registry-url https://$(acrLoginServer) \
                  --container-registry-user $(acrName) \
                  --container-registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv)
                # Set environment variables
                az webapp config appsettings set --name $(frontAppServiceQA) --resource-group $(ResourceGroupName) \
                  --settings API_URL="$(api_url_as_qa)" \
 ```

Nuevamente, esto crea un recurso dentro de Azure Portal.
<img width="757" alt="image" src="https://github.com/user-attachments/assets/7cda0752-01f0-4d5c-bdab-c6b33c22b5fb">

<img width="1511" alt="image" src="https://github.com/user-attachments/assets/654bde28-cf32-40a5-9daa-79d40dce4185">

### Paso 3
Se agregan las variables necesarias para esta etapa considerando que debe haber un entorno de QA y un entorno de Producción.

 ```
# Azure App Services
  AppServicePlanLinux: 'LinuxAppPlan01'

  # API (QA)
  backAppServiceQA: 'gaggio-as-crud-api-qa'
  api_url_as_qa: 'https://$(backAppServiceQA).azurewebsites.net/api/Employee'

  # FRONT (QA)
  frontAppServiceQA: 'gaggio-as-crud-front-qa'
  front_url_as_qa: 'https://$(frontAppServiceQA).azurewebsites.net'

  # API (Prod)
  backAppServiceProd: 'gaggio-as-crud-api-prod'
  api_url_as_prod: 'https://$(backAppServiceProd).azurewebsites.net/api/Employee'

  # FRONT (Prod)
  frontAppServiceProd: 'gaggio-as-crud-front-prod'
  front_url_as_prod: 'https://$(frontAppServiceProd).azurewebsites.net'
  
 ```

<img width="600" alt="image" src="https://github.com/user-attachments/assets/5bc0285b-23f5-411b-a51c-4066b70266fc">
<img width="423" alt="image" src="https://github.com/user-attachments/assets/cc082e35-b9ea-4540-a1a8-15ee12040f30">


<img width="1121" alt="image" src="https://github.com/user-attachments/assets/4a3af1c7-b2f2-40aa-a188-d84dd25ce9fc">

### Paso 4
Se agrega un nuevo job a la etapa para ejecutar las pruebas E2E de Cypress en el entorno de QA de los Azure App Service.
 ```

# -------------------------------------------------------------------------------
# |           RUN INTEGRATION TESTS ON AZURE APP SERVICES                       |
# -------------------------------------------------------------------------------
    - job: RunCypressTests
      displayName: 'Run Cypress Tests on App Services'
      dependsOn: [DeployFrontImageToAppServiceQA, DeployAPIImageToAppServiceQA]
      condition: succeeded()
      steps:
        - script: npm install ts-node typescript --save-dev
          displayName: 'Install typescript'
          workingDirectory: '$(frontPath)'

        - script: npx cypress run --env apiUrl=$(api_url_as_qa),homeUrl=$(front_url_as_qa)
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

<img width="664" alt="image" src="https://github.com/user-attachments/assets/1d80697b-a190-4fe8-adee-32a29cbf3005">

Paso 5
Se agrega una nueva etapa que depende de la etapa de despliegue a QA anterior. En esta etapa, se realiza un nuevo despliegue utilizando nuevamente Azure App Services pero en un entorno de producción, por lo que se modifican las variables de entorno y se requiere una aprobación manual antes de comenzar esta etapa.


 ```

# -------------------------------------------------------------------------------
# |     STAGE DEPLOY FRONTEND AND BACKEND TO AZURE APP SERVICES PROD             |
# -------------------------------------------------------------------------------
- stage: DeployImagesToAppServiceProd
  displayName: 'Deploy Docker Images to Azure App Service (Prod)'
  dependsOn: DeployImagesToAppServiceQA
  condition: succeeded()
  jobs:
# -------------------------------------------------------------------------------
# |               DEPLOY API TO AZURE APP SERVICE                               |
# -------------------------------------------------------------------------------
    - deployment: DeployAPIImageToAppServiceProd
      displayName: 'Deploy API to Azure App Service (Prod)'
      pool:
        vmImage: 'ubuntu-latest'
      environment: 'prod'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureCLI@2
                displayName: 'Create API Azure App Service resource and configure image'
                inputs:
                  azureSubscription: '$(ConnectedServiceName)'
                  scriptType: 'bash'
                  scriptLocation: 'inlineScript'
                  inlineScript: |
                    # Check if API App Service already exists
                    if ! az webapp list --query "[?name=='$(backAppServiceProd)' && resourceGroup=='$(ResourceGroupName)'] | length(@)" -o tsv | grep -q '^1$'; then
                      echo "API App Service doesn't exist. Creating new..."
                      # Create App Service without specifying container image
                      az webapp create --resource-group $(ResourceGroupName) --plan $(AppServicePlanLinux) --name $(backAppServiceProd) --deployment-container-image-name "nginx"  # Especifica una imagen temporal para permitir la creación
                    else
                      echo "API App Service already exists. Updating image..."
                    fi

                    # Configure App Service to use Azure Container Registry (ACR)
                    az webapp config container set --name $(backAppServiceProd) --resource-group $(ResourceGroupName) \
                      --container-image-name $(acrLoginServer)/$(backImageName):$(backImageTag) \
                      --container-registry-url https://$(acrLoginServer) \
                      --container-registry-user $(acrName) \
                      --container-registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv)
                    # Set environment variables
                    az webapp config appsettings set --name $(backAppServiceProd) --resource-group $(ResourceGroupName) \
                      --settings DBCONNECTIONSTRING="$(cnn_string_prod)" \

# -------------------------------------------------------------------------------
# |               DEPLOY FRONTEND TO AZURE APP SERVICE                           |
# -------------------------------------------------------------------------------
    - deployment: DeployFrontImageToAppServiceProd
      displayName: 'Deploy Front to Azure App Service (Prod)'
      pool:
        vmImage: 'ubuntu-latest'
      environment: 'prod'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureCLI@2
                displayName: 'Create Front Azure App Service resource and configure image'
                inputs:
                  azureSubscription: '$(ConnectedServiceName)'
                  scriptType: 'bash'
                  scriptLocation: 'inlineScript'
                  inlineScript: |
                    # Check if Front App Service already exists
                    if ! az webapp list --query "[?name=='$(frontAppServiceProd)' && resourceGroup=='$(ResourceGroupName)'] | length(@)" -o tsv | grep -q '^1$'; then
                      echo "Front App Service doesn't exist. Creating new..."
                      # Create App Service without specifying container image
                      az webapp create --resource-group $(ResourceGroupName) --plan $(AppServicePlanLinux) --name $(frontAppServiceProd) --deployment-container-image-name "nginx"  # Especifica una imagen temporal para permitir la creación
                    else
                      echo "Front App Service already exists. Updating image..."
                    fi

                    # Configure App Service to use Azure Container Registry (ACR)
                    az webapp config container set --name $(frontAppServiceProd) --resource-group $(ResourceGroupName) \
                      --container-image-name $(acrLoginServer)/$(frontImageName):$(frontImageTag) \
                      --container-registry-url https://$(acrLoginServer) \
                      --container-registry-user $(acrName) \
                      --container-registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv)
                    # Set environment variables
                    az webapp config appsettings set --name $(frontAppServiceProd) --resource-group $(ResourceGroupName) \
                      --settings API_URL="$(api_url_as_prod)" \
 ```
<img width="707" alt="image" src="https://github.com/user-attachments/assets/d6e85c15-be90-4981-ae75-81e2cfee9fe3">

Se muestran los recursos generados a partir de la ejecución de la presente etapa.
<img width="1511" alt="image" src="https://github.com/user-attachments/assets/36d4cb8a-7db0-4508-b68e-71e71d878bc7">
<img width="757" alt="image" src="https://github.com/user-attachments/assets/2ff6155d-6565-44d4-b787-c29d9dbf368e">
<img width="461" alt="image" src="https://github.com/user-attachments/assets/9c8737db-7b72-429e-b917-377ac0c4526c">
<img width="741" alt="image" src="https://github.com/user-attachments/assets/ec343838-aa70-4942-8adb-03b377ef7d7b">


### Paso 6
<img width="1190" alt="image" src="https://github.com/user-attachments/assets/f2199f10-4126-41bd-ba72-d70af9533809">
<img width="1190" alt="image" src="https://github.com/user-attachments/assets/57d3c1ae-e5bd-4eb1-943a-0bc5d20a2217">

![image](https://github.com/user-attachments/assets/e3b9a030-0d32-4551-816b-af3eecf74fcc)


<img width="1512" alt="image" src="https://github.com/user-attachments/assets/6f25adb3-bd24-4b01-a80f-eb7f92e9ac81">

Agregué la condición de publicar los test por mas que fallen luego de la clase del 22/10

<img width="1509" alt="image" src="https://github.com/user-attachments/assets/c4d52fb3-e6de-4d93-930b-541532bf2444">
