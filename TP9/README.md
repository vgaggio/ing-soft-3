
## 1- Desarrollo:
### Paso 1

Se agrega al pipeline una nueva etapa que depende de la etapa de construcción y prueba, y de la etapa de construcción y subida de las imágenes de Docker a ACR. Inicialmente, esta etapa crea un nuevo contenedor utilizando la imagen de la API y lo despliega en un Azure App Service, para lo cual se requiere un App Service Plan de Linux.



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

Imagen Paso 1

Paso 2
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

<img width="1511" alt="image" src="https://github.com/user-attachments/assets/654bde28-cf32-40a5-9daa-79d40dce4185">

Paso 3
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


<img width="1121" alt="image" src="https://github.com/user-attachments/assets/4a3af1c7-b2f2-40aa-a188-d84dd25ce9fc">

Paso 4
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
<img width="707" alt="image" src="https://github.com/user-attachments/assets/d6e85c15-be90-4981-ae75-81e2cfee9fe3">

Paso 5
Se agrega una nueva etapa que depende de la etapa de despliegue a QA anterior. En esta etapa, se realiza un nuevo despliegue utilizando nuevamente Azure App Services pero en un entorno de producción, por lo que se modifican las variables de entorno y se requiere una aprobación manual antes de comenzar esta etapa.

A su vez, se creó una nueva base de datos SQL para el entorno productivo, de modo que los datos almacenados entre QA y Prod son independientes. La base de datos inicial se utiliza para todos los despliegues en QA, mientras que la nueva será conectada a las aplicaciones productivas (en entorno Prod).

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

Se muestran los recursos generados a partir de la ejecución de la presente etapa.
<img width="1511" alt="image" src="https://github.com/user-attachments/assets/36d4cb8a-7db0-4508-b68e-71e71d878bc7">

Paso 6
Se entrega el pipeline completo que consta de las siguientes etapas:

BuildAndTest: construcción, pruebas unitarias, cobertura de código y análisis de código con SonarCloud.
DockerBuildAndPush: construcción de las imágenes de Docker y subida a ACR.
DeployToWebAppQA: despliegue del proyecto en Azure Web Apps (QA) y ejecución de pruebas de integración.
DeployToWebAppProd: despliegue del proyecto en Azure Web Apps (Prod), previa aprobación manual.
DeployToACIQA: despliegue del proyecto en Azure Container Instances (QA) y ejecución de pruebas de integración.
DeployToACIProd: despliegue del proyecto en Azure Container Instances (Prod), previa aprobación manual.
DeployImagesToAppServiceQA: despliegue del proyecto en Azure App Services (QA) y ejecución de pruebas de integración.
DeployImagesToAppServiceProd: despliegue del proyecto en Azure App Services (Prod), previa aprobación manual.
trigger:
- main

pool:
  vmImage: 'windows-latest'


variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'
  frontPath: 'EmployeeCrudAngular'
  backPath: 'EmployeeCrudApi'
  frontDrop: 'angular-drop'
  backDrop: 'dotnet-drop'

# Docker Images
  ResourceGroupName: 'IngSw3-UCC'
  ConnectedServiceName: 'AzureResourceManager'
  acrLoginServer: "$(acrName).azurecr.io"
  acrName: 'mcingsw3uccacr'
  backImageName: 'employee-crud-api'
  frontImageName: 'employee-crud-angular'

# Azure Web Apps

  # API (QA)
  backWebAppNameQA: 'Chattas-Backend-QA'
  api_url_wa_qa: 'https://$(backWebAppNameQA).azurewebsites.net/api/Employee'

  # Front (QA)
  frontWebAppNameQA: 'Chattas-Frontend-QA'
  front_url_wa_qa: 'https://$(frontWebAppNameQA).azurewebsites.net'

  # API (Prod)
  backWebAppNameProd: 'Chattas-Backend-Prod'
  api_url_wa_prod: 'https://$(backWebAppNameProd).azurewebsites.net/api/Employee'

  # Front (Prod)
  frontWebAppNameProd: 'Chattas-Frontend-Prod'
  front_url_wa_prod: 'https://$(frontWebAppNameProd).azurewebsites.net'

# Azure Container Instances

  # API (QA)
  backContainerInstanceNameQA: 'chattas-crud-api-qa'
  backImageTag: 'latest' 
  container-cpu-api-qa: 1
  container-memory-api-qa: 1.5
  api_url_aci_qa: "http://$(backContainerInstanceNameQA).brazilsouth.azurecontainer.io/api/Employee"

  # Front (QA)
  frontContainerInstanceNameQA: 'chattas-crud-front-qa'
  frontImageTag: 'latest' 
  container-cpu-front-qa: 1
  container-memory-front-qa: 1.5
  front_url_aci_qa: "http://$(frontContainerInstanceNameQA).brazilsouth.azurecontainer.io"

  # API (Prod)
  backContainerInstanceNameProd: 'chattas-crud-api-prod' 
  container-cpu-api-prod: 1
  container-memory-api-prod: 1.5
  api_url_aci_prod: "http://$(backContainerInstanceNameProd).brazilsouth.azurecontainer.io/api/Employee"

  # Front (Prod)
  frontContainerInstanceNameProd: 'chattas-crud-front-prod'
  container-cpu-front-prod: 1
  container-memory-front-prod: 1.5
  front_url_aci_prod: "http://$(frontContainerInstanceNameProd).brazilsouth.azurecontainer.io"

# Azure App Services
  AppServicePlanLinux: 'ASP-IngSw3UCC-Linux'

  # API (QA)
  backAppServiceQA: 'chattas-as-crud-api-qa'
  api_url_as_qa: 'https://$(backAppServiceQA).azurewebsites.net/api/Employee'

  # FRONT (QA)
  frontAppServiceQA: 'chattas-as-crud-front-qa'
  front_url_as_qa: 'https://$(frontAppServiceQA).azurewebsites.net'

  # API (Prod)
  backAppServiceProd: 'chattas-as-crud-api-prod'
  api_url_as_prod: 'https://$(backAppServiceProd).azurewebsites.net/api/Employee'

  # FRONT (Prod)
  frontAppServiceProd: 'chattas-as-crud-front-prod'
  front_url_as_prod: 'https://$(frontAppServiceProd).azurewebsites.net'

# -------------------------------------------------------------------------------
# |             STAGE BUILD AND TEST FRONTEND AND BACKEND                        |
# -------------------------------------------------------------------------------
stages:
- stage: BuildAndTest
  displayName: "Build and Test API and Front"
  jobs:
# -------------------------------------------------------------------------------
# |                        BUILD AND TEST API                                   |
# -------------------------------------------------------------------------------
  - job: BuildDotnet
    displayName: "Build and Test API"
    pool:
      vmImage: 'windows-latest'
    steps:
    - checkout: self
      fetchDepth: 0

    - task: DotNetCoreCLI@2
      displayName: 'Restore NuGet Packages'
      inputs:
        command: restore
        projects: '$(solution)'

    - task: DotNetCoreCLI@2
      displayName: 'Test API'
      inputs:
        command: 'test'
        projects: '**/*.Tests.csproj'
        arguments: '--collect:"XPlat Code Coverage"'

    - task: PublishCodeCoverageResults@2
      displayName: 'Publish API code coverage results'
      inputs:
        summaryFileLocation: '$(Agent.TempDirectory)/**/*.cobertura.xml'
        failIfCoverageEmpty: false

    - task: SonarCloudPrepare@2
      inputs:
        SonarCloud: 'SonarCloudBack'
        organization: 'maximo-chattas'
        scannerMode: 'MSBuild'
        projectKey: 'maximo-chattas_Dotnet_Project'
        projectName: 'maximo-chattas_Dotnet_Project'
        extraProperties:
          sonar.tests=EmployeeCrudApi.Tests/
          sonar.sources=$(backPath)
      
    - task: DotNetCoreCLI@2
      displayName: 'Compile API'
      inputs:
        command: build
        projects: '$(solution)'
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/api  --self-contained false'

    - task: DotNetCoreCLI@2
      displayName: 'Publish app'
      inputs:
        command: publish
        publishWebProjects: True
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/api'
        zipAfterPublish: true
      env:
        DBCONNECTIONSTRING: $(cnn_string_qa)
    
    - task: SonarCloudAnalyze@2
      inputs:
        jdkversion: 'JAVA_HOME_17_X64'
    
    - task: SonarCloudPublish@2
      inputs:
        pollingTimeoutSec: '300'

    - task: PublishBuildArtifacts@1
      displayName: 'Publish API artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: $(backDrop)
        publishLocation: 'Container'

    - task: PublishPipelineArtifact@1
      displayName: 'Publish API Dockerfile'
      inputs:
        targetPath: '$(Build.SourcesDirectory)/docker/api/dockerfile'
        artifact: 'dockerfile-back'

# -------------------------------------------------------------------------------
# |                        BUILD AND TEST FRONT                                  |
# -------------------------------------------------------------------------------
  - job: BuildAngular
    displayName: "Build and Test Angular"
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - checkout: self
      fetchDepth: 0
    - task: NodeTool@0
      displayName: 'Install Node.js'
      inputs:
        versionSpec: '22.x'
    
    - script: npm install
      displayName: 'Install dependencies'
      workingDirectory: $(frontPath)

    - script: npx ng test --karma-config=karma.conf.js --watch=false --browsers ChromeHeadless --code-coverage
      displayName: 'Test frontend'
      workingDirectory: $(frontPath)
      continueOnError: true  # Continue on test failure

    - task: PublishCodeCoverageResults@2
      displayName: 'Publish frontend code coverage results'
      inputs:
        summaryFileLocation: '$(frontPath)/coverage/lcov.info'
        failIfCoverageEmpty: false
      condition: always()

    - task: PublishTestResults@2
      displayName: 'Publish frontend unit tests results'
      inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: '$(frontPath)/test-results/test-results.xml'
        failTaskOnFailedTests: true
      condition: always()
    
    - task: SonarCloudPrepare@2
      inputs:
        SonarCloud: 'SonarCloudFront'
        organization: 'maximo-chattas'
        scannerMode: 'CLI'
        configMode: 'manual'
        cliProjectKey: 'maximo-chattas_Angular_Project'
        cliProjectName: 'maximo-chattas_Angular_Project'
        cliSources: $(frontPath)/src

    - script: npx ng build --output-path=dist/employee-crud-angular
      displayName: 'Compile frontend'
      workingDirectory: $(frontPath)
    
    - task: SonarCloudAnalyze@2
      inputs:
        jdkversion: 'JAVA_HOME_17_X64'
    
    - task: SonarCloudPublish@2
      inputs:
        pollingTimeoutSec: '300'

    - task: PublishBuildArtifacts@1
      displayName: 'Publish frontend artifacts'
      inputs:
        PathtoPublish: '$(Build.SourcesDirectory)/$(frontPath)/dist'
        ArtifactName: $(frontDrop)
    
    - task: PublishPipelineArtifact@1
      displayName: 'Publish frontend Dockerfile'
      inputs:
        targetPath: '$(Build.SourcesDirectory)/docker/front/Dockerfile'
        artifact: 'dockerfile-front'

# -------------------------------------------------------------------------------
# |     STAGE BUILD AND PUSH FRONTEND AND BACKEND DOCKER IMAGES TO ACR          |
# -------------------------------------------------------------------------------
- stage: DockerBuildAndPush
  displayName: 'Build and Push Docker Images to ACR'
  dependsOn: BuildAndTest
  jobs:
# -------------------------------------------------------------------------------
# |                        BUILD AND PUSH API                                    |
# -------------------------------------------------------------------------------
    - job: docker_build_and_push_api
      displayName: 'Build and Push API Docker Image to ACR'
      pool:
        vmImage: 'ubuntu-latest'
        
      steps:
        - checkout: self
        - task: DownloadPipelineArtifact@2
          displayName: 'Download API artifacts'
          inputs:
            buildType: 'current'
            artifactName: $(backDrop)
            targetPath: '$(Pipeline.Workspace)/$(backDrop)'
        - task: DownloadPipelineArtifact@2
          displayName: 'Download API Dockerfile'
          inputs:
            buildType: 'current'
            artifactName: 'dockerfile-back'
            targetPath: '$(Pipeline.Workspace)/dockerfile-back'

        - task: AzureCLI@2
          displayName: 'Log in to Azure Container Registry (ACR)'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              az acr login --name $(acrLoginServer)
    
        - task: Docker@2
          displayName: 'Build API Docker image'
          inputs:
            command: build
            repository: $(acrLoginServer)/$(backImageName)
            dockerfile: $(Pipeline.Workspace)/dockerfile-back/dockerfile
            buildContext: $(Pipeline.Workspace)/$(backDrop)
            tags: 'latest'

        - task: Docker@2
          displayName: 'Upload API Docker image to ACR'
          inputs:
            command: push
            repository: $(acrLoginServer)/$(backImageName)
            tags: 'latest'
  
# -------------------------------------------------------------------------------
# |                        BUILD AND PUSH FRONT                                 |
# -------------------------------------------------------------------------------
    - job: docker_build_and_push_front
      displayName: 'Build and Push Frontend Docker Image to ACR'
      pool:
        vmImage: 'ubuntu-latest'
        
      steps:
        - checkout: self

        - task: DownloadBuildArtifacts@1
          displayName: 'Download Frontend artifacts'
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: $(frontDrop)
            downloadPath: '$(System.ArtifactsDirectory)'
      
        - task: DownloadPipelineArtifact@2
          displayName: 'Download Frontend Dockerfile'
          inputs:
            buildType: 'current'
            artifactName: 'dockerfile-front'
            targetPath: '$(Pipeline.Workspace)/dockerfile-front'

        - task: AzureCLI@2
          displayName: 'Log in to Azure Container Registry (ACR)'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: bash
            scriptLocation: inlineScript
            inlineScript: |
              az acr login --name $(acrLoginServer)
    
        - task: Docker@2
          displayName: 'Build Frontend Docker image'
          inputs:
            command: build
            repository: $(acrLoginServer)/$(frontImageName)
            dockerfile: $(Pipeline.Workspace)/dockerfile-front/Dockerfile
            buildContext: $(System.ArtifactsDirectory)/$(frontDrop)/employee-crud-angular/browser
            tags: 'latest'

        - task: Docker@2
          displayName: 'Upload Frontend Docker image to ACR'
          inputs:
            command: push
            repository: $(acrLoginServer)/$(frontImageName)
            tags: 'latest'

# -------------------------------------------------------------------------------
# |       STAGE DEPLOY FRONTEND AND BACKEND TO AZURE WEB APPS QA                 |
# -------------------------------------------------------------------------------
- stage: DeployToWebAppQA
  displayName: 'Deploy to Azure Web APPs (QA)'
  dependsOn: BuildAndTest
  condition: succeeded()
  pool:
    vmImage: 'windows-latest'
    
  jobs:
# -------------------------------------------------------------------------------
# |               DEPLOY API TO AZURE WEB APP                                    |
# -------------------------------------------------------------------------------
  - job: DeployBackQA
    displayName: 'Deploy Backend to Azure Web APP (QA)'
    steps:
    - task: DownloadBuildArtifacts@1
      displayName: 'Download API artifacts'
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: $(backDrop)
        downloadPath: '$(System.ArtifactsDirectory)'

    - task: AzureRmWebAppDeployment@4
      displayName: 'Deploy API To Azure Web App'
      inputs:
        ConnectionType: 'AzureRM'
        azureSubscription: 'Azure subscription 1(f7ecd1b3-7c3e-4cb1-9653-fc24ede1711b)'
        appType: 'webApp'
        WebAppName: $(backWebAppNameQA)
        packageForLinux: '$(System.ArtifactsDirectory)/$(backDrop)/**/*.zip'
        AppSettings: '-DBCONNECTIONSTRING "$(cnn_string_qa)"'

# -------------------------------------------------------------------------------
# |               DEPLOY FRONT TO AZURE WEB APP                                 |
# -------------------------------------------------------------------------------
  - job: DeployFrontQA
    displayName: 'Deploy Frontend to Azure Web APP (QA)'
    steps:
    - task: DownloadBuildArtifacts@1
      displayName: 'Download Frontend artifacts'
      inputs:
        buildType: 'current'
        downloadType: 'single'
        artifactName: $(frontDrop)
        downloadPath: '$(System.ArtifactsDirectory)'
    
    - script: |
        echo window['env'] = { apiUrl: '$(api_url_wa_qa)' }; > $(System.ArtifactsDirectory)/$(frontDrop)/employee-crud-angular/browser/assets/env.js
      displayName: 'Set env.js content'
    
    - task: AzureRmWebAppDeployment@4
      displayName: 'Deploy Frontend To Azure Web App'
      inputs:
        ConnectionType: 'AzureRM'
        azureSubscription: 'Azure subscription 1(f7ecd1b3-7c3e-4cb1-9653-fc24ede1711b)'
        appType: 'webApp'
        WebAppName: $(frontWebAppNameQA)
        packageForLinux: '$(System.ArtifactsDirectory)/$(frontDrop)/employee-crud-angular/browser'
  
# -------------------------------------------------------------------------------
# |              RUN INTEGRATION TESTS ON AZURE WEB APPS                         |
# -------------------------------------------------------------------------------
  - job: RunCypressTests
    displayName: 'Run Cypress Tests on Web Apps'
    dependsOn: [DeployFrontQA, DeployBackQA]
    condition: succeeded()
    steps:
      - script: npm install ts-node typescript --save-dev
        displayName: 'Install typescript'
        workingDirectory: '$(frontPath)'

      - script: npx cypress run --env apiUrl=$(api_url_wa_qa),homeUrl=$(front_url_wa_qa)
        workingDirectory: '$(frontPath)'
        displayName: 'Run integration tests'
        continueOnError: true
      - task: PublishTestResults@2
        inputs:
          testResultsFormat: 'JUnit'
          testResultsFiles: '*.xml'
          searchFolder: '$(frontPath)/cypress/results'
          testRunTitle: 'Cypress Integration Tests'

# -------------------------------------------------------------------------------
# |       STAGE DEPLOY FRONTEND AND BACKEND TO AZURE WEB APPS PROD                 |
# -------------------------------------------------------------------------------
- stage: DeployToWebAppProd
  displayName: 'Deploy to Azure Web APPs (Prod)'
  dependsOn: DeployToWebAppQA
  condition: succeeded()
  pool:
    vmImage: 'windows-latest'
    
  jobs:
# -------------------------------------------------------------------------------
# |               DEPLOY API TO AZURE WEB APP                                    |
# -------------------------------------------------------------------------------
  - deployment: DeployBack
    displayName: 'Deploy Backend to Azure Web APP (Prod)'
    environment: 'prod'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadBuildArtifacts@1
            displayName: 'Download API artifacts'
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: $(backDrop)
              downloadPath: '$(System.ArtifactsDirectory)'

          - task: AzureRmWebAppDeployment@4
            displayName: 'Deploy API To Azure Web App'
            inputs:
              ConnectionType: 'AzureRM'
              azureSubscription: 'Azure subscription 1(f7ecd1b3-7c3e-4cb1-9653-fc24ede1711b)'
              appType: 'webApp'
              WebAppName: $(backWebAppNameProd)
              packageForLinux: '$(System.ArtifactsDirectory)/$(backDrop)/**/*.zip'
              AppSettings: '-DBCONNECTIONSTRING "$(cnn_string_prod)"'

# -------------------------------------------------------------------------------
# |               DEPLOY FRONT TO AZURE WEB APP                                 |
# -------------------------------------------------------------------------------
  - deployment: DeployFront
    displayName: 'Deploy Frontend to Azure Web APP (QA)'
    environment: 'prod'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadBuildArtifacts@1
            displayName: 'Download Frontend artifacts'
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: $(frontDrop)
              downloadPath: '$(System.ArtifactsDirectory)'
          
          - script: |
              echo window['env'] = { apiUrl: '$(api_url_wa_prod)' }; > $(System.ArtifactsDirectory)/$(frontDrop)/employee-crud-angular/browser/assets/env.js
            displayName: 'Set env.js content'
          
          - task: AzureRmWebAppDeployment@4
            displayName: 'Deploy Frontend To Azure Web App'
            inputs:
              ConnectionType: 'AzureRM'
              azureSubscription: 'Azure subscription 1(f7ecd1b3-7c3e-4cb1-9653-fc24ede1711b)'
              appType: 'webApp'
              WebAppName: $(frontWebAppNameProd)
              packageForLinux: '$(System.ArtifactsDirectory)/$(frontDrop)/employee-crud-angular/browser'

# -------------------------------------------------------------------------------
# |      STAGE DEPLOY FRONTEND AND BACKEND TO AZURE CONTAINER INSTANCES QA      |
# -------------------------------------------------------------------------------
- stage: DeployToACIQA
  displayName: 'Deploy to Azure Container Instances (QA)'
  dependsOn: DockerBuildAndPush
  jobs:
# -------------------------------------------------------------------------------
# |               DEPLOY API TO AZURE CONTAINER INSTANCES                       |
# -------------------------------------------------------------------------------
    - job: deploy_api_to_aci_qa
      displayName: 'Deploy API to Azure Container Instances (QA)'
      pool:
        vmImage: 'ubuntu-latest'
      steps:
        - task: AzureCLI@2
          displayName: 'Deploy Docker Image to ACI'
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
                --environment-variables DBCONNECTIONSTRING="$(cnn_string_qa)" \
                --restart-policy Always \
                --cpu $(container-cpu-api-qa) \
                --memory $(container-memory-api-qa)

# -------------------------------------------------------------------------------
# |               DEPLOY FRONT TO AZURE CONTAINER INSTANCES                      |
# -------------------------------------------------------------------------------
    - job: deploy_front_to_aci_qa
      displayName: 'Deploy Frontend to Azure Container Instances (QA)'
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

# -------------------------------------------------------------------------------
# |           RUN INTEGRATION TESTS ON AZURE CONTAINER INSTANCES                |
# -------------------------------------------------------------------------------
    - job: RunCypressTests
      displayName: 'Run Cypress Tests on Container Instances'
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


# -------------------------------------------------------------------------------
# |    STAGE DEPLOY FRONTEND AND BACKEND TO AZURE CONTAINER INSTANCES PROD       |
# -------------------------------------------------------------------------------
- stage: DeployToACIProd
  displayName: 'Deploy to Azure Container Instances (Prod)'
  dependsOn: DeployToACIQA
  condition: succeeded()
  jobs:
# -------------------------------------------------------------------------------
# |               DEPLOY API TO AZURE CONTAINER INSTANCES                       |
# -------------------------------------------------------------------------------
    - deployment: deploy_api_to_aci_prod
      displayName: 'Deploy API to Azure Container Instances (Prod)'
      pool:
        vmImage: 'ubuntu-latest'
      environment: 'prod'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureCLI@2
                displayName: 'Deploy Docker Image to ACI'
                inputs:
                  azureSubscription: '$(ConnectedServiceName)'
                  scriptType: bash
                  scriptLocation: inlineScript
                  inlineScript: |
                    echo "Resource Group: $(ResourceGroupName)"
                    echo "Container Instance Name: $(backContainerInstanceNameProd)"
                    echo "ACR Login Server: $(acrLoginServer)"
                    echo "Image Name: $(backImageName)"
                    echo "Image Tag: $(backImageTag)"
                    echo "Connection String: $(cnn_string_qa)"
                
                    az container delete --resource-group $(ResourceGroupName) --name $(backContainerInstanceNameProd) --yes

                    az container create --resource-group $(ResourceGroupName) \
                      --name $(backContainerInstanceNameProd) \
                      --image $(acrLoginServer)/$(backImageName):$(backImageTag) \
                      --registry-login-server $(acrLoginServer) \
                      --registry-username $(acrName) \
                      --registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv) \
                      --dns-name-label $(backContainerInstanceNameProd) \
                      --ports 80 \
                      --environment-variables DBCONNECTIONSTRING="$(cnn_string_prod)" \
                      --restart-policy Always \
                      --cpu $(container-cpu-api-prod) \
                      --memory $(container-memory-api-prod)

# -------------------------------------------------------------------------------
# |               DEPLOY FRONT TO AZURE CONTAINER INSTANCES                      |
# -------------------------------------------------------------------------------
    - deployment: deploy_front_to_aci_prod
      displayName: 'Deploy Frontend to Azure Container Instances (Prod)'
      pool:
        vmImage: 'ubuntu-latest'
      environment: 'prod'
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureCLI@2
                displayName: 'Deploy Docker Image to ACI'
                inputs:
                  azureSubscription: '$(ConnectedServiceName)'
                  scriptType: bash
                  scriptLocation: inlineScript
                  inlineScript: |
                    echo "Resource Group: $(ResourceGroupName)"
                    echo "Container Instance Name: $(frontContainerInstanceNameProd)"
                    echo "ACR Login Server: $(acrLoginServer)"
                    echo "Image Name: $(frontImageName)"
                    echo "Image Tag: $(frontImageTag)"
                
                    az container delete --resource-group $(ResourceGroupName) --name $(frontContainerInstanceNameProd) --yes

                    az container create --resource-group $(ResourceGroupName) \
                      --name $(frontContainerInstanceNameProd) \
                      --image $(acrLoginServer)/$(frontImageName):$(frontImageTag) \
                      --registry-login-server $(acrLoginServer) \
                      --registry-username $(acrName) \
                      --registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv) \
                      --dns-name-label $(frontContainerInstanceNameProd) \
                      --ports 80 \
                      --environment-variables API_URL="$(api_url_aci_prod)" \
                      --restart-policy Always \
                      --cpu $(container-cpu-front-prod) \
                      --memory $(container-memory-front-prod)

# -------------------------------------------------------------------------------
# |     STAGE DEPLOY FRONTEND AND BACKEND TO AZURE APP SERVICES QA              |
# -------------------------------------------------------------------------------
- stage: DeployImagesToAppServiceQA
  displayName: 'Deploy Docker Images to Azure App Service (QA)'
  dependsOn: 
  - BuildAndTest
  - DockerBuildAndPush
  condition: succeeded()
  jobs:
# -------------------------------------------------------------------------------
# |               DEPLOY API TO AZURE APP SERVICE                               |
# -------------------------------------------------------------------------------
    - job: DeployAPIImageToAppServiceQA
      displayName: 'Deploy API to Azure App Service (QA)'
      pool:
        vmImage: 'ubuntu-latest'
      steps:
        - task: AzureCLI@2
          displayName: 'Create API Azure App Service resource and configure image'
          inputs:
            azureSubscription: '$(ConnectedServiceName)'
            scriptType: 'bash'
            scriptLocation: 'inlineScript'
            inlineScript: |
              # Check if API App Service already exists
              if ! az webapp list --query "[?name=='$(backAppServiceQA)' && resourceGroup=='$(ResourceGroupName)'] | length(@)" -o tsv | grep -q '^1$'; then
                echo "API App Service doesn't exist. Creating new..."
                # Create App Service without specifying container image
                az webapp create --resource-group $(ResourceGroupName) --plan $(AppServicePlanLinux) --name $(backAppServiceQA) --deployment-container-image-name "nginx"  # Especifica una imagen temporal para permitir la creación
              else
                echo "API App Service already exists. Updating image..."
              fi

              # Configure App Service to use Azure Container Registry (ACR)
              az webapp config container set --name $(backAppServiceQA) --resource-group $(ResourceGroupName) \
                --container-image-name $(acrLoginServer)/$(backImageName):$(backImageTag) \
                --container-registry-url https://$(acrLoginServer) \
                --container-registry-user $(acrName) \
                --container-registry-password $(az acr credential show --name $(acrName) --query "passwords[0].value" -o tsv)
              # Set environment variables
              az webapp config appsettings set --name $(backAppServiceQA) --resource-group $(ResourceGroupName) \
                --settings DBCONNECTIONSTRING="$(cnn_string_qa)" \

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



<img width="1130" alt="image" src="https://github.com/user-attachments/assets/10d295ae-b88a-47fe-8542-4bdb16381aa7">

Las pruebas unitarias (de API y Front) y las pruebas de integración (en los 3 despliegues) son aprobadas.

Imagen 2

El análisis de Sonar Cloud es aprobado.

Imagen 3

Se tiene el reporte de cobertura de código del proyecto.

Imagen 4
