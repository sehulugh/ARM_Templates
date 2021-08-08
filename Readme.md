## Createing ARM Templates
---
### Powershell Scripts
[Docs](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)

Connect to Azure Account
```powershell
Connect-AzAccount
```

Set Subscription
```powershell
Set-AzContext [SubscriptionID/SubscriptionName]
```

Create Resource Group
```powershell
New-AzResourceGroup `
  -Name myResourceGroup `
  -Location "Central US"
```

Deploy Template
```powershell
$templateFile = "C:\Users\hulughsx\OneDrive - Reed Elsevier Group ICO Reed Elsevier Inc\Apps\LIA\ARM_Templates\azuredeploy.json"
New-AzResourceGroupDeployment `
  -Name blanktemplate `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
```
| Command | Description |
| -------- | ------------|
| `New-AzResourcegroupDeployment` | Command to deploy to a resource group |
| `Name` | any name you want to use to identify the deployment |
| `ResourceGroupName` | Resource group name previosly created |
| `TemplateFIle` | path to template json file, typically saved ina variable $variableName |


### Add Resource
**Snippet**
```json
"resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "sesughdemostorage",
            "location": "eastus",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2",
            "properties": {
                "supportsHttpsTrafficOnly": true
                }
        }
    ]
```
**Powershell script**
```powerShell
New-AzResourceGroupDeployment `
  -Name sesughdemostorage `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
```
### Add parameters {#addParameters}
Makes templates reusable by allowig you use placehlders for fields like resource name
Parameter is added with the syntax `"[parameters('parameterName')]"`
Default values are used when not specified in the deploy script
**Snippet**
```json
"parameters":{
        "storageName": {
            "type": "string",
            "minLength":3,
            "maxLength":24
        },
        "storageSKU": {
            "type": "string",
            "defaultValue":"Standard_LRS",
            "allowedValues": [
                "Standard_LRS",
                "Standard_GRS",
                "Standard_RAGRS",
                "Standard_ZRS"
            ]
        },
    },
  "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[parameters('storageName')]",
            "location": "eastus",
            "sku": {
              "name": "[parameters('storageSKU')]"
            },
            ...
        }
    ]
```
**Powershell script**
```powerShell
New-AzResourceGroupDeployment `
  -Name sesughdemostorage `
  -ResourceGroupName myResourceGroup `
  -TemplateFile $templateFile
  -storageName newStorageName
```
> **Resource Updates**
When you deploy a template and the resources already exist, no change is made to those resources, if a property has changed, it is updated but existing files in the resource are not affected
### Add template functions
Template functions are used to dynamically construct values, parameters used previously is also an inbuilt function, you can also create UDFs. ([see how](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/user-defined-functions))
Below we use a function to get the resourcegroup's location and use it for our resource
**Snippet**
```json
"parameters":{
        "storageName": {
            "type": "string",
            "minLength":3,
            "maxLength":24
        },
        "storageSKU": {
            "type": "string",
            "defaultValue":"Standard_LRS",
            "allowedValues": [
                "Standard_LRS",
                "Standard_GRS",
                "Standard_RAGRS",
                "Standard_ZRS"
            ]
        },
        "location": {
            "type": "string",
            "defaultValue":"[resourceGroup().location]"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2019-04-01",
            "name": "[parameters('storageName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('storageSKU')]"
            },
            "kind": "StorageV2",
            "properties": {
                "supportsHttpsTrafficOnly": true
                }
        }
    ]
```
Powershell script same as in [Add Paramteters](#addParameters) section
### Add Variables
