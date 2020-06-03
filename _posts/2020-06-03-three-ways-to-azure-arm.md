---
layout: post
title: "Three ways to Azure ARM"
tags: [Azure, ARM, PowerShell]
---

In latest months I switched from developing applications to designing their build infrastructure on Microsoft [Azure](https://azure.microsoft.com/en-us/). It was quite challenging at the beginning, but it's a very interesting and rewarding field.

The process of provisioning resources on Azure is called **ARM** and can be completely automated.

Despite I'm still learning everyday about this wide and fascinating environment, there's a couple of interesting things I'd like to share.

##### Architecture

I was unable to find a part of Microsoft documentation that clearly states that every interaction with ARM (and DevOps too) is achieved by a **REST APIs**.

Digging into the [Az module](https://github.com/Azure/azure-powershell) repository on GitHub, you'll see a wide usage of [Azure SDK](https://github.com/Azure/azure-sdk-for-net) through all **CmdLet**s. As stated in the README the SDK directly mirrors Azure service's **REST** endpoints. For what I've seen a good part of this library is created with automatic code generation.

**Az module** is the successor of [Azure RM](https://docs.microsoft.com/en-us/powershell/azure/azurerm/overview?view=azurermps-6.13.0) that it's no more extended from 2018 and it will not receive bug fixes after December 2020. See also [this article](https://azure.microsoft.com/es-es/blog/azure-powershell-cross-platform-az-module-replacing-azurerm/) for more informations.

At end of the day the only way to interact with ARM is through its REST API interface. Staying at a higher level we can use Az module, [Az CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) and [ARM templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview).

##### Az module

Az module should be your first choice to interact with ARM. It's a cross-[PowerShell](https://docs.microsoft.com/en-us/powershell/) module that allow you to do _almost_ anything. In my experience, despite it's still limited, I'm pretty sure you'll be unable to automate some step using Az module APIs. Sometime this may happens for lack of documentation or simply because it's simply not possible. I also experimented weird behaviours where the same exact input values makes an Az module call to fail, but perfectly works for Az CLI.

At the moment I was developing ARM scripts Az module was missing [AAD](https://azure.microsoft.com/it-it/services/active-directory/) commands. I'm sure these still don't exist at moment of publishing of this article. See this [GitHub issue](https://github.com/Azure/azure-powershell/issues/9074) about that. The solution was to rely on Az CLI.

It follows a sample snippet of code that initializes an **App Service**:
```powershell
$subscriptionId = '************************************'
$group = 'your-res-group'
$location = 'West Europe'
$name = 'your-app-service'
$plan = 'your-app-service-plan'
$properties = @{
    alwaysOn = $true
    $appProperties.serverFarmId = "/subscriptions/$subscriptionId/resourceGroups/$group/providers/Microsoft.Web/serverfarms/$plan"
}
New-AzResource -ResourceGroupName $group -Location $location -ResourceName $name -ResourceType 'Microsoft.Web/sites' `
               -Kind $kind -Properties $properties -Force | Out-Null
```

I said _initializes_ because you'll need to configure more things such as [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing), AAD and problably other settings.

##### Az CLI

**Az CLI** is a terminal application written in Python that allows ARM interaction using a command line interface. I personally found its documentation a bit better than the one of Az module.

Staying on **AAD** matter, you can use [az webapp auth](https://docs.microsoft.com/en-us/cli/azure/webapp/auth?view=azure-cli-latest) to configure AAD (and generally authentication and authorization) settings of an App Service. Az CLI is organized in sub commands, the first is normally relative to a domain (such as `webapp` for App Service or `image` Virtual machines images) while the second represents an operation such as `show`, `list`, `update` and many others.

In the following snippet (to stay on topic) I'll show you how to set ADD configuration for an App Service:
```powershell
$clientId = '************************************'
$clientSecret = '********************************'
$group = 'your-res-group'
$name = 'your-app-service'
$allowedTokens = @('api://********-****-****-****-************')
"az webapp auth update --name $name --resource-group $group --enabled true" +
    " --action LoginWithAzureActiveDirectory --aad-client-id $clientId --aad-client-secret $clientSecret" +
    " --token-store true --aad-allowed-token-audiences $($allowedTokens -join ' ') | Out-Null" | Invoke-Expression
```

Execution of `az` command returns to standard output a JSON text of the altered or inspected resources.

##### ARM Templates

ARM Templates are **JSON** files that describe resource(s) to be updated. This is how looks an ARM template that provision an **App Service**, exported by Visual Studio 2019 (with few changes to parameters names):
```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "name": {
      "type": "string"
    },
    "location": {
      "type": "string"
    },
    "plan": {
      "type": "string"
    },
    "group": {
      "type": "string"
    }
  },
  "resources": [
    {
      "location": "[parameters('location')]",
      "name": "[parameters('name')]",
      "type": "Microsoft.Web/sites",
      "apiVersion": "2015-08-01",
      "tags": {
        "[concat('hidden-related:', resourceId(parameters('group'),'Microsoft.Web/serverfarms', parameters('plan')))]": "empty"
      },
      "kind": "app",
      "properties": {
        "name": "[parameters('name')]",
        "kind": "app",
        "httpsOnly": true,
        "reserved": false,
        "serverFarmId": "[resourceId(parameters('group'),'Microsoft.Web/serverfarms', parameters('plan'))]"
      },
      "identity": {
        "type": "SystemAssigned"
      }
    }
  ]
}
```

As you can see it's possible to make an ARM template parametric. It's allowed to use expressions to combine parameters values. Details of the overall syntax are described in this [document](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-syntax).

[Azure DevOps](https://azure.microsoft.com/en-us/services/devops/) allows you to export a whole resource group or an App Service definition instead of creating it. You can download a template or copy it to clipbloard, but recently it has been added a feature to store templates in a **Template Library**. There are also other means to generate a template file.

To deploy a template you'll need a call to [New-AzResourceGroupDeployment](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-powershell) of Az module:
```powershell
$group = 'your-res-group'
$templateParams = @{
    name = 'your-app-service'
    location = 'West Europe'
    plan = 'your-plan'
    group = $group
}

New-AzResourceGroupDeployment -ResourceGroupName $group -TemplateFile 'C:\ARM\template.json' -TemplateParameterObject $templateParams
```

You can accomplish the same exact task using Az CLI. In that case you'll need to use the `create` sub command of [az group deployment](https://docs.microsoft.com/en-us/cli/azure/group/deployment?view=azure-cli-latest):
```powershell
$group = 'your-res-group'
az group deployment create --resource-group $group --template-file 'C:\ARM\template.json' \
    --parameters name=your-app-service location='West Europe' plan=your-plan group=$group
```

In both cases there are many options in the way you can supply template or parameters, but further deepening would be beyond the scope of this article.

As more as you came confident, you can load an ARM template, modify JSON nodes and save it to a temporary file to feed `New-AzResourceGroupDeployment` or `az group deployment create`. Or if you prefer you can directly generate it from scratch. In the latter case, since I've experimented problems with PowerShell built-in JSON functions (_due to not fixed bugs_), I'll suggest to use a third party library like Newtonsoft [Json.NET](https://www.newtonsoft.com/json).

You can't send again the same template with changes in update mode, you'll need to destroy the resource before.

##### Pipelines

The way I've designed resources provisioning is through PowerShell scripts invoked via a [PowerShell@2](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/utility/powershell?view=azure-devops) task configured into a **DevOps** release pipeline.

I created a script for each specific resource type and feed it with an XML definition. At the moment it's needed to copy and paste the minified source directly into the release variables, but we're evalutating to stage these files into the repository itself.

##### Terraform

In matter of resources provisioning I can't avoid mentioning [Terraform](https://www.terraform.io/). This interesting tool allows you to define resources with its [configuration language](https://www.terraform.io/docs/configuration/index.html) or through [JSON](https://www.terraform.io/docs/configuration/syntax-json.html). Then this system can take care of creating, updating or removing resources in [idempotency](https://www.youtube.com/watch?v=UJp6bCkdrwY) way.

You can opt to use it directly as a [CLI](https://www.terraform.io/docs/cli-index.html) application or use HashiCorp [Cloud](https://www.terraform.io/docs/cloud/index.html).

I'm still studying it, so I can't going deep too much (moreover would be beyond the scope of this article). Anyway I can show you a snippet of code to create an App Service:
```
provider "azurerm" {
  version         = "=1.44.0"
  subscription_id = "************************************"
  tenant_id       = "************************************"
  client_id       = "************************************"
  client_secret   = "********************************"
  skip_provider_registration = true
}

resource "azurerm_app_service" "your_app_service" {
  name                = "your-app-service"
  location            = "West Europe"
  resource_group_name = "your-res-group"
  app_service_plan_id = "your-plan"

  site_config {
      dotnet_framework_version = "v4.0"
      scm_type                 = "LocalGit"
  }
}
```

##### Conclusion

I suggest you to provision resources starting with a combination of Az module and Az CLI (giving priority to the first) and left ARM templates as last resort. I hope that sharing my experiences with Azure ARM could help you save a nice amount of time. _So, it's time to sculpt some cloud!_

I want to thank  my actual employer [Dev4Side](https://www.dev4side.com/), a **Gold Microsoft Partner**, for allowing me to work in a such interesting field.