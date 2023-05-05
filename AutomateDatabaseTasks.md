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
  
## SQL Server maintenance plan

Typical activities scheduled for SQL Server maintenance:

- Database and transaction log backups
- Database consistency checks
- Index maintenance
- Statistics updates

Database consistency checks, also known as CHECKDB (DBCC CHECKDB command) is the only way to check an entire database for corruption. You may performe these activities nightly. Commonly in production systems, the maintenance operations are spread out over the course of week, as both index maintenance and consistency checks are very I/O intensive. 

SQL Server offers a buit-in way to manage these tasks using Maintenance Plans. It creates a workflow of tasks to support databases. Maintenance plans are created as Integration Services packages, which allow you to schedule maintenance activities. It's also possible to use open-source scripts to perform database maintenance, to allow for more flexibility and control of maintenance activities.

### Best practices for maintenance plans

Maintenancee plans provide option to prune data from ```msdb``` databas, which actis as the data store for the SQL Server Agent. Maintenance plans also allow you to specify that older database backups should be removed from disk. Cleaning old backups helps by reducing the size of your backups volume and helps manage the size of the ```msdb``` database. Ensure that your backup retention period is longer than your consistency check. The backup operation will not detect corruption in a database, so it's possible to have corruption within a backup file. Maintenance plan activities are scheduled as SQL Agent jobs for execution.

### Create a maintenance plan

You can create a maintenance plan using SSMS. Multiple maintenance tasks can be combined in one maintenance plan. The best practice is to create a maintenance plan for each type of task - and possibly even for a specific database.

You need to specify a name for your maintenance plan, and you need to specify a run-as account. Most maintenance operations will run as the SQL Server Angent service account, but you may need to back up to a file share that only a specific account has access to, you would need to run as a different user. This is known as proxy user.

### Proxy account

A proxy account is an account with stored credentials that can be used by the SQL Server Agent to execute steps of a job as a specific user. The login information for this user is stored as a credential in the SQL Server instance.

### Job schedules

Job schedules are a component of the job system in the ```msdb``` system database. SQL Agent jobs and schedules have a many-to-many relationship. The maintenance plan does not allow creation of independent schedules. It creates a specific schedule for each maintenance plan.

The options for maintenance tasks are:

- Check Database integrity - executes DBCC ChECKDB, to validate the contets of each database page to ensure its logical and physical consistnecy.
- Shrink Database - reduces the size of database or transaction log file by moving data into free space on pages. When enough space is consumed, the free space can be returned to the file system. It's recommended that you never execute this action as part of regular maintenance as it leads to severe index fragmentation which can harm database performance. The operation itself is very I/O and CPU intensive and severely impact system performance
- Reorganize/Rebuild Index - will check the level of fragmentation in a database's indexes, and either rebuild or reorganize the index based on level of fragmentation. Rebuilding index updates the statistics
- Update Statistics - updates the column and index statistics used by SQL Server to build query execution plans. This task allows to choose which tables and indexes are scanned, and the percentage or number of rows scanned.
- Cleanup History - deletes history of backups and restore operations from ```msdb``` database, as well as the history of SQL Server Agent jobs. This task is used to manage the size of ```msdb``` database.
- Execute SQL Server Agent Job - is used to execute a user-defined SQL Server Agent job
- Backup Database (Full/Differential/Log) - is used to back up databases. Full backup backs up the entire database. Differential backups back ups the pages in the database that have changed since the last full backup. Transaction log backup back up the active pages in your transaction log. Transaction log backup cannot be performed on databases in SIMPLE recovery mode.
- Maintenance Cleanup Tasks - remove old files related to maintenance plans, including text reports from maintenance execution, and backup files. It only removes backups on files in the folders specified.

Each task has a scope of user database, system database, or custem selection of databases. Each task has its hown specific configuration options.

### Multi-server automation

Ina multi-server environment, SQL Server Agent provides the option of designating one server as  master server that can execute jobs on other servers, designated as target servers. The master stores a master copy of the jobs and distributes the jobs to the target servers. Target servers connect to the master server periodically to update their schedule jobs.

### Task status notifications

One important part of automation is providing notifications in the event of job failure or if certain system errors are encountered. Alerting is most commonly done via email using the Database Mail functionality of SQL Server

Agent objects used in notifications are:

- Operators - alias for people or group who receives notifications
- Notifications - notify an operator of the completion, success or failure of a job.
- Alerts - assigned to an operator, for either a notification or a defined error condition

#### Operators

Act as an alias for user or group of users that have been configured to receive notifications of job completion, or to be informed of alerts have been sent to error log. An operator is defined as an operator name and contact information. To send email to an operator, you need to enable the email profile of the SQL Server Agent.

#### Notifications
You have the option of sending a notification on job completion, failure or success. Notification have a dependency on an operator existing in order to send a notification

#### Alerts

Alerts allow you to be proactive with monitoring. The agent reads the SQL Server error log and when it finds an error number for which an alert has been defined, it notifies an operator. In addition to monitoring the SQL Server error log, you can set up alerts to monitor SQL Server Performance conditions, as well as Windows Management Instrumentation (WMI) events. You can specify alerts to be raised in response to one or more events. A common pattern is to raise an alert on all SQL Server errors of level 16 and higher, and then add alerts for specific event types related to critical storage errors or Availability Group failover. Another example would be to alert on performance conditions such high CPU utilization or Low Page Life Expectancy.

You have option for how to respond to performance condition - you can notify and operator via email, or you can execute another SQL Server Agent job, which could resolve the problem. Executing another job is most commonly used in the scenario where the condition is well-known, and easily handled.

## Elastic jobs

Elastic jobs allow to run a set of T-SQL scripts against a collection of servers or databases as a one-time job or by using a defined schedule. Elastic jobs work similarly to SQL Server Agent jobs, except that they're limited to executing T-SQL. The jobs work across all tiers of Azure SQL Database.

To configure Elastic jobs, you need a Job agent and a database dedicated to managing you jobs. The recommended service tier is  S1 or higher.

elastic jobs components:

- Elastic Job agent - Azure resource for running and managing jobs.
- Job database - database deticated to manage your jobs
- Target group - collection of servers, elastic pools, and single databases in which a job will be run
- Job - one or more T-SQL scripts that compose a job step

If a server o elastic pool is the target, a credential within the master database of the server or pool should be created so that the job agent can enumerate the databases. For a single database, a database credential is all that is needed.

You can create an elastic job agent through the Azure portal, on the Elastic Job agent page.

You can create a target group by using either PowerShell or T-SQL
```PowerShell
# create MyServerGroup target group
$serverGroup = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'MyServerGroup'

$serverGroup | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName
```

Create elast job and add jobs steps:
```PowerShell
Write-Output "Creating a new job..."
$jobName = "MyFirstElasticJob"
$job = $jobAgent | New-AzSqlElasticJob -Name $jobName -RunOnce

Write-Output "Creating job steps for $($jobName) job..."
$sqlText1 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('MyTable')) CREATE TABLE [dbo].[MyTable]([Id] [int] NOT NULL);"

$job | Add-AzSqlElasticJobStep -Name "Step1" -TargetGroupName $serverGroup.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText1
```
run elastic job:
```PowerShell
Write-Output "Start the job..."
$jobExecution = $job | Start-AzSqlElasticJob
$jobExecution
```

### Use case scenarios

- Automate management tasks to run on a specific schedule
- Deploy schema changes
- Data movements
- Collect, and aggregate data for reporting or other purposes
- Load data from Azure Blob storage
- Configure jobs to execute across a collection of databases on a recurring basis
- Data processing over a large number of databases, for instance, telemetry collection. Results are then compiled into a single destination table for further analysis.
  
## Azure Automation

Azure Function and Logic Apps are both services that enable serverless workloads. Both services create workflows that are a collection of steps to execute complex tasks.

For more complete and granularity of automation, Azure Automation allows for process automation, configuration management, full integration with Azure platform options (RBAC and AAD), and can manage Azure and on-premises resources.

One of the unique benefits of Azure Automation is that it can manage resources within Azure or on-premises.

### Azure Automation components

Azure Automation supports both automation and configuration management activities. The components of Azure Automation needed to execute automated tasks:

- Runbook - are the unit of execution in Azure Automation. can be defined as one of three types: graphycal runbook based on PowerShell, a PowerShell script, or Python script
- Modules - defines an execution context for the PowerShell or Python code executing in runbook. In order to execute you code, you need to import the supporting modules.
- Credentials - store sensitive information that runbooks or configurations can use at runtime
- Schedules - are linked to runbooks and trigger a runbook at a specific time.

### Azure policy

Group Policies, or GPOs have been used by Windows server administrators for a long time, to manage security, provide consistency across the environment.
Azure Policy includes initiatives definitions to help estabilish and maintain compliance with different security standards for your Automation account. A policy provides a level of governance over Azure subscriptions. Policy can enforce rules and controls over Azure resources. Some examples might use are limiting the regions you can deploy a resource, enforcing naming standards, controlling resource sizes. You can use policies provided by Azure or specify custom policies using JSON.

Policies are assigned to a specific scope, which could be a management group, a subscription, a resource group, or individual resource. Individual policies can be groupd using a structure known as initiatives, sometimes called policy sets.

### Azure subscriptions and tags

Organizations use multiple subscriptions for several reasons, including budget management, security, or isolation of resources.

Tags are simply metadata that are used to better describe Azure resources. These tags are stored as key:value pairs and appear in the Azure portal associated with your Azure resources. When you use PowerShell or Azure CLI commands, you can filter commands based on tags.

```PowerShell
$rg=(get-AzResourceGroup)

$rg=($rg|where-object {($_.tags['Use'] -ne 'Internal')}).ResourceGroupName
```

Azure supports applying up to 15 tags to each resource.

Tags are also included in billing information, so tagging by cost center means it can be easier for management to break down charges.

```PowerShell
$tags = @{"Dept"="Finance"; "Status"="Normal"}

$resource = Get-AzResource -Name demoStorage -ResourceGroup demoGroup

New-AzTag -ResourceId $resource.id -Tag $tags
```
```
az resource tag --tags 'Dept=IT' 'Environment=Test' -g examplegroup -n examplevnet `

 --resource-type "Microsoft.Network/virtualNetworks"
 ```

 Tags are stored as plain text. Never add sensitive values to tags.

 ### Automation runbook

 In order to build an automation runbook, you need to first create an atumation account.

 Before you create you runbook, you'll import modules into your Automation account, to do this import, navigate to the Shared Resources section of the main blade for autotamion account, and select Modules Gallery.

 Next, you can optionally create a credential .

 On the Process Automation section, you Automation Account, select Runbooks, to create a runbook.

 To create a runbook, you must provide a name, the type of runbook, the runtime version, and optionally a description.

 After you've completed your code in the portal, select Test pane. This allows you to test your code in the context of Azure Automation. This allows you to separate PowerShell errors from error that might be generated from the context of automation execution.

 Hybrid runbooks are used when you need to execute cmdlets inside of a virtual machine. You'll need a configuration on the virtual machine and in the Azure Automation account.

 After you've successfully tested you runbook, you can Publish. A runbook must be published in order to be executed by the Azure service. After you've published the runbook, you can create a schedule in the Shared Resources.

 The default settings are for there to be no recurrence of the job.

 Once you created a schedule you can link to a runbook selecting Link to schedule in the runbook page.

 ## Automatie database workflows using Logic Apps

 Azure Logic Apps is a cloud-based platform for creating and running automated workflows that integrate your apps, data, services and systems. As member of Azure Integration Services, Logic Apss simplifies the way to connect legacy, modern, and cutting-edge systems across cloud, on-premises, and hybrid environments.

 The following list describes examle of tasks you can automate using Logic Apps service:

 - Schedule and send email notifications using Office 365 when a specific event happns
 - Route and process customer orders across on-premises systems
 - Move uploaded files from FTP server to AZure Storage
 - Monitar tweets, analyze the sentiment, and create alerts or tasks for items that need review.

Logic Apps integration platform provides prebuilt Microsoft-managed API connectors and built-in operations so you can connect and integrate apps, data, services and systems more easily.

You can create code snippets using Azure Functions and run that code from your workflow.

You can monitor, route, and publish events using Azure Event Grid

Logic Apps is fully managed by Microsoft Azure. These services automatically scale to meet your needs.

### SQL Server connector

SQL Server connector allows you to access SQL databases. You can then create automated workflows that are triggered by events in your database or other systems.

SQL Server connector supports the following editions:

- SQL Server
- Azure SQL Database
- Azure SQL Managed Instance

SQL Server connector requires that your tables contain data so that SQL connector oporations can return results when called.

If you want to start your workflow with a SQL Server trigger operation, you have to start with a blank workflow

SQL Server connector is available for logic apps in multi-tenant Azure Logic Apps, integration service environment (ISE), and single-tenant Azure Logic Apps.

- Consumption workflows in multi-tenant Azure Logic Apps - this connector is available only as a managed connector.
- Consumption workflows in an integration service environment - this connector is available as a managed connector and as an ISE connector that's designed to run in an ISE
- Standard workflows in single-tenant Azure Logic Apps - this connector is available as a managed connector and as a built-in connector that's designed to run in the same process as the single-tenant Azure Logic Apps runtime. The built-in version differs in the following ways:
  - The buitl-in connector has no triggers
  - The built-in connector has only one operation: Execute Query

### Logic App workflow

- Add a SQL Server trigger
- Add a SQL Server action
- Connect to Azure SQL Database

## Monitor automated tasks

It's important to monitor automated tasks to make sure they're working properly.

### Monitor runbooks

Runbooks should be modular, with logic that can be reused and restarted. Monitoring progress in a runbook ensures that the logic is executed correctly.

A database, storage account, or shared file can be used to monitor a runbook's progress. Ensure that your runbook checks the last action performed before initiating the next action. Based on the results the logic can either skip or continue

### Alerts

You can create metric alerts to monitor execution of runbooks. Alerts allow you to define conditions to monitor and actions to take  when conditions are met.

### Activity log

Runbooks are executed and details are collected in an activity log.

### Log Analytics

Azure Automation sed runbook job status and job streams to your Log Analytics workspace.

Azure Monitor logs integrated with Automation account enables you to

- View the status of your Automation jobs
- Write advanced queries across your job workflow
- Trigger an email or alert on your runbook job status
- Correlate data from multiple Automation jobs

Befire start using Log Analytics to query Automation jobs data, you must configure Diagnostic settings for your Automation Account.

### Monitor elastic job

With elastic jobs, job execution are logged, and any failures are automatically retried.

You can monitor elastic jobs execution through Azure portal, PowerShell, and T-SQL

```PowerShell
# get the latest 5 executions run
$jobAgent | Get-AzSqlElasticJobExecution -Count 5

# get the job step execution details
$jobExecution | Get-AzSqlElasticJobStepExecution

# get the job target execution details
$jobExecution | Get-AzSqlElasticJobTargetExecution -Count 2
```

```SQL
--View all execution status for all jobs
SELECT * 
FROM jobs.job_executions
ORDER BY start_time DESC;

--View all execution statuses for job named 'MyJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'MyJob'
ORDER BY start_time DESC;

-- View all active executions
SELECT * 
FROM jobs.job_executions
WHERE is_active = 1
ORDER BY start_time DESC;
```

