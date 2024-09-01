---
title: Azure Databricks y Azure DevOps
date: 2024-08-22
description: Databricks conexion entre Azure DevOps
tags:
  - Azure
  - Databricks
  - Cloud
categories:
  - Azure
  - AzureDevOps
  - Proyecto
dg-publish: true
draft: "false"
weight: 2
image: banner.png
---

## Que es Databricks

Para empezar, Databricks es una herramienta de analisis de datos que en pocas palabras te permite crear clusters rapidamente para analizar, y procesar grandes cantidades de datos y soporta Python, R, Scala etc.

## Por que deberiamos conectarlo contra Azure Devops?


## Laboratorio : Conexion contra Azure DevOps


Si quisieramos pasar de ambientes automaticamente, sin dejar asociado ningun nombre de un usuario como owner de los archivos, mantener todo con IaC en un repositorio, deberiamos realizar la conexion contra Azure DevOps.

### Requisitos:

- Databricks
- Service Principal
- Self Hosted or Azure Hosted


### Codigo:

[Pruebas en Postman](https://www.postman.com/mission-participant-26477422/workspace/databricks/collection/34475567-12da3afe-034f-4c48-a506-bd8277edfd7f)


En este proceso vamos a tener que utilizar varios tokens para poder hacer un "Clone Repo" (Basicamente clonar los archivos del repositorio de AzureDevOps hacia cualquier Databricks) en nombre de una app registration en la carpeta de Repos/ y no en la carpeta de Shared ni en una personal como suele suceder si copiamos manualmente utilizando el usuario

La primera parte del pipeline se vera asi:

~~~YAML
trigger:
- pruebadevops

pool:
  vmImage: ubuntu-latest

variables:
- group: TestFinal

steps:
- task: UsePythonVersion@0
  displayName: '1. Use Python 3.7'
  inputs:
    versionSpec: '3.7'
    addToPath: true
    architecture: 'x64'

- bash: |
   python -m pip install --upgrade pip
   python -m pip install msal
   python -m pip install databricks_cli
  displayName: '2. Install modules Python'

#We need to login as the service principal in order to obtain the token for the required services
- bash: |
    az login --service-principal -u $(client_id) -p $(client_pass) --tenant $(AZURE-TENANT-ID)
  displayName: '3. Az Login as service principal'

##-----------------------------------------------------------------------------------------##
##-----------------------------------------------------------------------------------------##
##-----------------------------------------------------------------------------------------##

#In this step, we obtain the token for AAD 
- bash: |
   response_aad=$(curl --location --request POST 'https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/token' \
   --header 'Content-Type: application/x-www-form-urlencoded' \
   --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAAAg9H7QEAAAD47t_dDgAAAIeFAvEBAAAA-vHf3Q4AAAA' \
   --data-urlencode 'grant_type=client_credentials' \
   --data-urlencode 'client_id=$(client_id)' \
   --data-urlencode 'client_secret
   =$(client_pass)' \
   --data-urlencode 'resource=https://management.core.windows.net/')
       
   access_token=$(echo $response_aad | jq -r '.access_token')
   
   #This command is to pass variables through the pipeline in different stages 
   echo "##vso[task.setvariable variable=access_token]$access_token"
    
  displayName: '4. Get-AAD-Token '

#Step to obtain Databricks token
- bash: |
   LOGIN_URL="https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/v2.0/token"
   
   response_data2=$(curl --location --request POST "$LOGIN_URL" \
   --header 'Content-Type: application/x-www-form-urlencoded' \
   --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAA' \
   --data-urlencode "grant_type=client_credentials" \
   --data-urlencode "client_id=$(client_id)" \
   --data-urlencode "client_secret=$(client_pass)" \
   --data-urlencode "scope=2ff814a6-3304-4ab8-85cb-cd0e6f879c1d/.default")
   
   token_data=$(echo "$response_data2" | jq -r '.access_token')
   
   echo "##vso[task.setvariable variable=token_data]$token_data"
  displayName: '5. Token_Data'

#Step to obtain Azure DevOps token (PAT, personal access token)
- bash: |
   LOGIN_URL="https://login.microsoftonline.com/$(azure-tenant-id)/oauth2/v2.0/token"
   
   # Make POST request to get access token
   response_ado=$(curl --location --request POST "$LOGIN_URL" \
   --header 'Content-Type: application/x-www-form-urlencoded' \
   --header 'Cookie: x-ms-gateway-slice=estsfd; stsservicecookie=estsfd; fpc=AmbU18E-FeJDrvAnHAneXPxfCtJrAQAAAOju390OAAAA' \
   --data-urlencode "grant_type=client_credentials" \
   --data-urlencode "client_id=$(client_id)" \
   --data-urlencode "client_secret=$(client_pass)" \
   --data-urlencode "scope=499b84ac-1321-427f-aa17-267ca6975798/.default")
   
   # Extract access token from response
   token_ado=$(echo "$response_ado" | jq -r '.access_token')
   

   echo "##vso[task.setvariable variable=token_ado]$token_ado"
  displayName: '6. Token_ADO PAT'

~~~

Primero haremos login con nuestra app registration, como ven, requiere tener un secret value:

~~~YAML
 az login --service-principal -u $(client_id) -p $(client_pass) --tenant $(AZURE-TENANT-ID)
 ~~~

 Luego obtendremos varios tokens:
 - AD Token: Del Azure Portal
 - ADO Token: De AzureDevOps (Mejor conocido como PAT o Personal Access Token)
 - Databricks Token: Necesario para poder interactuar con la API de Databricks

> 📝 **NOTA:** **Es necesario que nuestra app registration tenga permisos de Cluster Creation y de Service Manager dentro de nuestro Databricks, esto se hace desde adentro del Databricks en Settings**

Luego pasaremos a la segunda parte del pipeline, donde interactuamos con la API de Databricks para realizar el clonado.

El orden es el siguiente:

1. Crear credencial por cada repo que creemos (Una unica vez por repo, luego empezaremos a actualizarla para poder clonar cada vez que hagamos cambios, de lo contrario no nos dejara)
2. Obtener ID Credential, como tenemos que actualizar cada vez que corre el pipeline es necesario tener el ID
3. Updatear ID Credential
4. Clonar Repositorio 

~~~YAML
- task: Bash@3
  inputs:
    targetType: 'inline'
    script: |
      echo "Token Data: $(TOKEN_DATA)"
      echo "Token ADO: $(TOKEN_ADO)"
      echo "Token AAD: $(ACCESS_TOKEN)"
##-----------------------------------------------------------------------------------------##
##-----------------------------------------------------------------------------------------##
##-----------------------------------------------------------------------------------------##
#- task: Bash@3
#  inputs:
#    targetType: 'inline'
#    script: |
#      HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
#      URL="$(DATABRICKS_HOST)/api/2.0/git-credentials"
#      curl -v "${HEADERS[@]}" -X POST "$URL" --data-raw '{
#          "personal_access_token": "${TOKEN_ADO}",
#          "git_provider": "azureDevOpsServices",
#          "git_username": "UsernameAppRegistration"
#      }'
#  displayName: '6.1. Create Git Credential'


#Since we already had a credential, we need to get the ID number
- bash: |
   HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
   URL="${DATABRICKS_HOST}/api/2.0/git-credentials"
   
   response_credential=$(curl --location --request GET "$URL" \
   --header "Authorization: Bearer $TOKEN_DATA" \
   --header "Content-Type: application/json" \
   --data-raw '')
   echo "Response: $response_credential"
   git_credential=$(echo $response_credential | jq -r '.credentials[0].credential_id')
   echo "THEY SAID $git_credential"
   echo "##vso[task.setvariable variable=git_credential]$git_credential"
  displayName: '7. Get Credentials'

#Step to update the credential
- bash: |
   HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json")
   URL="${DATABRICKS_HOST}/api/2.0/git-credentials/${GIT_CREDENTIAL}"
   
   response_update=$(curl --location --request PATCH "$URL" \
   --header "Authorization: Bearer $TOKEN_DATA" \
   --header "Content-Type: application/json" \
   --data-raw '{
     "personal_access_token": "$(TOKEN_ADO)",
     "git_username": "UsernameAppRegistration",
     "git_provider": "azureDevOpsServices"
   }' )
   echo "$response_update"
  displayName: '8. UpdateGitCredential'

#Step to Create Repo using databricks API
- bash: |
   URL="$(DATABRICKS_HOST)/api/2.0/repos"
   RepoURL="https://dev.azure.com/{OrgName}/{ProjectName}/_git/{RepositorioName}"
   
   HEADERS=(-H "Authorization: Bearer ${TOKEN_DATA}" -H "Content-Type: application/json" -H "X-Databricks-Azure-SP-Management-Token: ${ACCESS_TOKEN}" -H "X-Databricks-Org-Id: 4388658498039786" -H "X-Databricks-Azure-Workspace-Resource-Id: /subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/RG/providers/Microsoft.Databricks/workspaces/DatabricksName")
   
   curl -v "${HEADERS[@]}" -X POST "$(DATABRICKS_HOST)/api/2.0/repos" --data-raw '{
       "personal_access_token": "${TOKEN_ADO}",
       "url" : "https://dev.azure.com/{OrgName}/{ProjectName}/_git/{RepositorioName}",
       "provider": "azureDevOpsServices",
       "path": "/Repos/Test/",
       "sparse_checkout": {
           "patterns": [
             "parent-folder/child-folder"
           ]
         }
   }'
  displayName: '8- Clone Repo /Create Repo copy copy'



- publish: $(System.DefaultWorkingDirectory)
  artifact: CodigoProyecto
  ~~~

  Para tener en cuenta, en este pipeline utilice una Libreria de Variables:

  - azure-tenant-id
  - Databricks-Host
  - ORG-ID
