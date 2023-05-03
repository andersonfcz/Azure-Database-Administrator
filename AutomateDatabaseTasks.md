# Automate deployment of database

## Describe deployment models

Azure Resource Manager templates are able to deploy a full set o resources in one single declarative template. You can build dependencies ito the templates and using parameters to change deployment values at deployment time. The benefit of these deployments is that they use a declarative model, which defines what should be created, the ARM framework then determines how to deploy it. The alternative to the declarative model is the imperative model. which follow a prescriptive order of tasks to be executed.

## Azure Resource Manager templates

ARM templates allow to create and deploy an entire infrastructure in a declarative framework. Resource Manager supports orchestration, which manages the deployment of interdependent resources so that they're created in the correct order. ARM templates support extensibility, which allows to run PowerShell or Bash scripts on your resources after they're deployed.

### PowerShell

Provides a core module known as Az Powershell module. PowerShell is more commonly used for resource modification and status retrieval, although it's possible to create resources using it. PowerShell can be used to deploy ARM templates, so in a sense it supports both declarative and imperative models.

### Azure CLI

Is similar to PowerShell in that it can be used either imperatively or declaratively, it provides a mechanism to deploy or modify resources. Some commands for Azure PostgreSQL and MySQL DB are only available in Azure CLI.

### Azure portal

Is a graphical interface to Azure Resource Manager. Any resources you build and deploy using the portal will  have an ARM template that you can capture by selecting Export template in the Overview section of resource group

### Azure DevOps

In Azure DevOps, deployments are carried out using Azure Pipelines, which is a fully featured continuous integration and continuous delivery that allows to automate the build, testing and deployment of your code.
You can deploy resources in two ways. Calling a PowerShell script, or defining tasks that stage your artifact and then deploys the templates.

### Continuous integration

Is a development methodology that focused on making small changes to code and frequent code check-ins to the version control system. CI provides an automated way to build, package, and test applications.

## Automate deployment by using Bicep

Automating database deployment is a crucial ability to create a reliable and sustainable development process.

### ARM template

Is a JavaScript Object Notation (JSON) documont that describe the resources to deploy within an Azure Resource Group. Uses a declarative model.

#### Benefits

- Repeatable - are idempotent, which allows you to repeatedly deploy your infrastructure and have confidence your resources are deployed in a consistent manner.
- Orchestration - take care of the complexities of ordering operations for deployments and when possible will deploy in parallel.
- Modular - can be split and combined at will
- Exportable code - allow you to easily recreate your environment for DR or documentation purposes
- Authoring tools - can be authored using VSCode template tool extension. It provides intellisens, syntax highlighting, in-line help, and many other functions.

#### Deploy an ARM template with PowerShell

You can scope your deployment when using PowerShell. You can deploy to a resource group, a subscription, a Management Group, or a tenant.

ARM template definition to create a single DB

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "_generator": {
      "name": "bicep",
      "version": "0.5.6.12127",
      "templateHash": "17606057535442789180"
    }
  },
  "parameters": {
    "serverName": {
      "type": "string",
      "defaultValue": "[uniqueString('sql', resourceGroup().id)]",
      "metadata": {
        "description": "The name of the SQL logical server."
      }
    },
    "sqlDBName": {
      "type": "string",
      "defaultValue": "SampleDB",
      "metadata": {
        "description": "The name of the SQL Database."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "administratorLogin": {
      "type": "string",
      "metadata": {
        "description": "The administrator username of the SQL logical server."
      }
    },
    "administratorLoginPassword": {
      "type": "secureString",
      "metadata": {
        "description": "The administrator password of the SQL logical server."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2021-08-01-preview",
      "name": "[parameters('serverName')]",
      "location": "[parameters('location')]",
      "properties": {
        "administratorLogin": "[parameters('administratorLogin')]",
        "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
      }
    },
    {
      "type": "Microsoft.Sql/servers/databases",
      "apiVersion": "2021-08-01-preview",
      "name": "[format('{0}/{1}', parameters('serverName'), parameters('sqlDBName'))]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard",
        "tier": "Standard"
      },
      "dependsOn": [
        "[resourceId('Microsoft.Sql/servers', parameters('serverName'))]"
      ]
    }
  ]
}
```

This file can be deployed from a URI:

```PowerShell
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter an Azure location (i.e. centralus)"
$adminUser = Read-Host -Prompt "Enter the SQL server administrator username"
$adminPassword = Read-Host -Prompt "Enter the SQL server administrator password" -AsSecureString

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/quickstarts/microsoft.sql/sql-database/azuredeploy.json" -administratorLogin $adminUser -administratorLoginPassword $adminPassword
```

### Bicep

Is a declarative language that allows to deploy Azure resources. It's commonly described as an Infrastructure-as-Code tool.

#### Benefits

- Continuous full support - provides support for all resource type and API versions for Azure services, which means that as soon as a resource provider introduces new resources types and API versions, you can use them in your Bicep file.
- Simple syntax - comparad to an JSON file it'll be more concise and easier to read
- Easy to use - requires no previous knowledge of programming languages and is easy to write and understand

```
param location string = resourceGroup().location
param storageAccountName string = 'toylaunch${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-06-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

#### Bicep vs. JSON

```JSON
{
  "type": "Microsoft.Sql/servers",
  "apiVersion": "2021-11-01-preview",
  "name": "string",
  "location": "string",
  "tags": {
    "tagName1": "tagValue1",
    "tagName2": "tagValue2"
  },
  "identity": {
    "type": "string",
    "userAssignedIdentities": {}
  },
  "properties": {
    "administratorLogin": "string",
    "administratorLoginPassword": "string",
    "administrators": {
      "administratorType": "ActiveDirectory",
      "azureADOnlyAuthentication": "bool",
      "login": "string",
      "principalType": "string",
      "sid": "string",
      "tenantId": "string"
    },
    "federatedClientId": "string",
    "keyId": "string",
    "minimalTlsVersion": "string",
    "primaryUserAssignedIdentityId": "string",
    "publicNetworkAccess": "string",
    "restrictOutboundNetworkAccess": "string",
    "version": "string"
  }
}
```

```
resource symbolicname 'Microsoft.Sql/servers@2021-11-01-preview' = {
  name: 'string'
  location: 'string'
  tags: {
    tagName1: 'tagValue1'
    tagName2: 'tagValue2'
  }
  identity: {
    type: 'string'
    userAssignedIdentities: {}
  }
  properties: {
    administratorLogin: 'string'
    administratorLoginPassword: 'string'
    administrators: {
      administratorType: 'ActiveDirectory'
      azureADOnlyAuthentication: bool
      login: 'string'
      principalType: 'string'
      sid: 'string'
      tenantId: 'string'
    }
    federatedClientId: 'string'
    keyId: 'string'
    minimalTlsVersion: 'string'
    primaryUserAssignedIdentityId: 'string'
    publicNetworkAccess: 'string'
    restrictOutboundNetworkAccess: 'string'
    version: 'string'
  }
}
```

#### Deploying an Azure SQL DB using Bicep with PowerShell

```
@description('The name of the SQL logical server.')
param serverName string = uniqueString('sql', resourceGroup().id)

@description('The name of the SQL Database.')
param sqlDBName string = 'SampleDB'

@description('Location for all resources.')
param location string = resourceGroup().location

@description('The administrator username of the SQL logical server.')
param administratorLogin string

@description('The administrator password of the SQL logical server.')
@secure()
param administratorLoginPassword string

resource sqlServer 'Microsoft.Sql/servers@2021-08-01-preview' = {
  name: serverName
  location: location
  properties: {
    administratorLogin: administratorLogin
    administratorLoginPassword: administratorLoginPassword
  }
}

resource sqlDB 'Microsoft.Sql/servers/databases@2021-08-01-preview' = {
  parent: sqlServer
  name: sqlDBName
  location: location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
}
```

To deploy this file, save it as main.bicep on your local computer and run:

```PowerShell
New-AzResourceGroup -Name exampleRG -Location eastus
New-AzResourceGroupDeployment -ResourceGroupName exampleRG -TemplateFile ./main.bicep -administratorLogin "<admin-login>"
```

### Source control for templates

ARM templates and Bicep files are an example of IaC. It's important to protect and version this code.

### Automate deployment by using PowerShell

PowerShell is a cross-platform command shell that provides features for automation. It can accept and return both text and .NET objects

PowerShell provides a core module known as Az PowerShell which is a set of cmdlets to manage resources directly from PowerShell.

#### Az.Sql PowerShell module

Is a subset of Az PowerShell that allows to manage Azure SQL resources.

Can be used however you use PowerShell including PowerShellGet, Azure Cloud Shell and Az PowerShell Docker container.

The syntax follows the structure verb-noun:

```PowerShell
<command-name> -<Required Parameter Name> <Required Parameter Value>
[-<Optional Parameter Name> <Optional Parameter Value>]
[-<Optional Switch Parameters>]
[-<Optional Parameter Name>] <Required Parameter Value>
```

Commands always begin with a command name, such as ```Get-AzSqlServer```, which returns information about one or more servers. The command name is then followed by a parameter name, then it's followed with a parameter value

```PowerShell
Get-AzSqlServer -ResourceGroupName "ResourceGroup01" -ServerName "Server01"
```
```PowerShell
New-AzSqlDatabase -ResourceGroupName "ResourceGroup01" -ServerName "Server01" -DatabaseName "Database01"
```
```PowerShell
New-AzSqlInstance -Name managedInstance2 -ResourceGroupName ResourceGroup01 -ExternalAdminName DummyLogin -EnableActiveDirectoryOnlyAuthentication -Location westcentralus -SubnetId "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/resourcegroup01/providers/Microsoft.Network/virtualNetworks/vnet_name/subnets/subnet_name" -LicenseType LicenseIncluded -StorageSizeInGB 1024 -VCore 16 -Edition "GeneralPurpose" -ComputeGeneration Gen4

$val = Get-AzSqlInstance -Name managedInstance2 -ResourceGroupName ResourceGroup01 -ExpandActiveDirectoryAdministrator
```

### Automate deployment by using Azure CLI

Database autamation is becoming a necessity for businesses of all sizes. It will provide the following benefits:

- Granular control of an application or database
- Easy to scale, improving efficiency when dealing with large numbers of assets
- Ability to reuse scripts
- Facilitates troubleshooting tasks

Azure CLI is a cross-platform command-line tool that allows you to create and manage resources. You can run commands through the terminal using cli or scripts

You can install on Linux, Mac or Windows. Run it from a browser using Cloud Shell on Azure portal or inside a Docker container.

Azure CLI follows the syntax ```reference name``` - ```comand``` - ```parameter``` - ```parameter value```
```
az account set --subscription "my subscription name"
```

#### PowerShell vs. Azure CLI

The main difference between is the shell environments that they support

| Shell Environment | Azure CLI | Azure PowerShell |
| ----------------- | --------- | ---------------- |
| Cmd | Yes | |
| Bash | Yes | |
| Windows PowerShell | Yes | Yes |
| PowerShell | Yes | Yes |

| Command | Azure CLI | Azure PowerShell |
| ------- | --------- | ----- |
| Sign in with Web Browser | az login | Connect-AzAccount |
| Get available subscriptions | az account list | Get-AzSubscription |
| Set Subscription | az account set -subscription | Set-AzContext -Subscription |
| List VM | az vm list | Get-AzVM |
| Create a SQL Server | as sql server create | New-AzSqlServer |

#### Deploying SQL DB using Azure CLI

Example of how to deploy SQL Db and create a firewall rule using Azure CLI:

```
let "randomIdentifier=$RANDOM*$RANDOM"

$resourceGroup = "<your resource group>"
$location = "<your location preference>"
$server = "dp300-sql-server-$randomIdentifier"
$login = "sqladmin"
$password = "Pa$$w0rD-$randomIdentifier"

az sql server create --name $server --resource-group $resourceGroup --location "$location" --admin-user $login --admin-password $password

az sql server firewall-rule create --resource-group $resourceGroup --server $server -n AllowYourIp --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

### Deploying ARM template using Azure CLI and PowerShell

ARM templates are parameterized, and you will need to pass in parameters, either inline or through the use of a parameter file

```
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
 -TemplateFile c:\MyTemplates\azuredeploy.json `
 -TemplateParameterFile c:\MyTemplates\storage.parameters.json
 ```

 The parameter and template file can be stored in a Git repo, Azure Blob Storage or any place where it's accessible from the deploying machine.

 ```
 az deployment group create --resource-group ExampleResourceGroup --template-file '\path\template.json'
 ```

 To deploy remoti linked templates with relative path that are stored in a storage account, use query-string to specify the SAS token

 ```
 az deployment group create \
  --name linkedTemplateWithRelativePath \
  --resource-group myResourceGroup \
  --template-uri "https://stage20210126.blob.core.windows.net/template-staging/mainTemplate.json" \
  --query-string $sasToken
  ```

  Currently Azure CLI doesn't support deploying remote Bicep files. You can use Bicep CLI to build the Bicep file to a JSON template and then load the JSON to the remote location

  You can review your deployed resources using
  ```
  az resource list --resource-group exampleRG
  ```
  