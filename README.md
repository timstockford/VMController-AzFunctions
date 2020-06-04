# VMController - Azure Functions

## Overview

Basic proof of concept collection of Powershell functions to Start, Stop & Query status of an Azure VM on demand.

This differs to other VM Controller functions as there is no timer built in, and is designed to be called:

- Directly via URL, with API key
- Via an application UI, e.g. PowerApp or Webpage

---

## TL;DR

Functions url takes following parameters:

| Param          | Desc                                                                                     | Example                                             |
|----------------|------------------------------------------------------------------------------------------|-----------------------------------------------------|
| code           | Authentication API key for the deployed Azure function (not required if testing locally) | code=jihfi763hrf8d3hd89j2x927ehwd8x                 |
| SubscriptionId | Azure Subscription Id where the VM is hosted                                                                 | SubscriptionId=6baed530-da79-4597-9bdf-817c5e1036ce |
| ResourceGroup  | Name of Resource Group the VM resides in                                                 | ResourceGroup=MyVMResources                         |
| VMName         | Azure Portal name for VM                                                                 | VMName=computer1                                    |

>- All parameters are required
>- Ensure pre-requisite Managed Identity has access to the VM's.  See [Security](#grant-the-managed-service-identity-contributor-access-to-the-subscription-or-resource-group-so-it-can-perform-actions) section for more info.

Example URL:

```http
https://<servername>/api/GetVMStatus?code=jihfi763hrf8d3hd89j2x927ehwd8x&SubscriptionId=6baed530-da79-4597-9bdf-817c5e1036ce&ResourceGroup=MyVMResources&VMName=computer1
```

---

## Pre-requisities

Install the following tools locally:

- Install [Azure Core Tools version 2.x](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local)
- Install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

This project was written with Visual Studio Code, leveraging the following add-ins:

- [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)
- [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

Optionally install the [Azure local storage emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator) to avoid debug runtime errors.

> The local functions server will download any dependancies defined for the functions, e.g. in this case the *az powershell module* when the server starts.

### **Create a new resource group and function application on Azure**

Run the following PowerShell command and specify the value for the function application name in the TemplateParameterObject hashtable.

```powershell
New-AzResourceGroup -Name <resource group name> -Location <location>

New-AzResourceGroupDeployment -ResourceGroupName <resource group name> -TemplateParameterObject @{"functionAppName" = "<your function app name>"} -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-functions-managed-identity/azuredeploy.json" -verbose
```

This should create a new resource group with a function application and a managed service identity enabled. The id of the service principal for the MSI should be returned as an output from the deployment.

Example: principalId    String   cac1fa06-2ad8-437d-99f6-b75edaae2921

### **Grant the managed service identity contributor access to the subscription or resource group so it can perform actions**

The below command sets the access at the subscription level.

```powershell
$Context = Get-AzContext
New-AzRoleAssignment -ObjectId <principalId> -RoleDefinitionName Contributor -Scope "/subscriptions/$($Context.Subscription)"
```

### Clone repository or download files to local machine

- Download the repository files or clone to local machine.
- Change to the PowerShell/VMController directory.

### Get the local.settings.json values from the function application created in Azure

```powershell
func azure functionapp fetch-app-settings <function app name>
```

This should create a local.settings.json file in the VMController directory beside the host.json with the settings from the Azure function app.

### **Test the functions locally**

Start the function with the following command

```powershell
func start
```

You can then call a trigger function by performing a post against the function on the admin api. Open up another Powershell console session and run:

With Powershell:

```powershell
Invoke-RestMethod "https://localhost:7071/api/GetVMStatus?SubscriptionId=6baed530-da79-4597-9bdf-817c5e1036ce&ResourceGroup=MyVMResources&VMName=computer1" -Method post -Body '{}' -ContentType "application/json"
```

With curl:

```dos
curl "http://localhost:7071/api/GetVMStatus?SubscriptionId=6baed530-da79-4597-9bdf-817c5e1036ce&ResourceGroup=MyVMResources&VMName=computer1"
```

---

## Publish the functions to the function application in Azure

```powershell
func azure functionapp publish <function app name>
```

---

## Known Issues

- When running for the first time, the functions server will download any module dependancies (in this case the 'az' powershell module).  In some instances this download has been blocked by the desktop anti-virus client, but the only error displayed is from the function server with an "Access Denied" in the log.  To avoid this, add an exclusion to the anti-virus client for the local functions app working directory, usually *C:\Users\username\AppData\Local\AzureFunctions*.

---

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

---

## Acknowledgments

- This project & documentation was based on [eamonoreilly/StartStopPowerShellFunction](https://github.com/eamonoreilly/StartStopPowerShellFunction)
