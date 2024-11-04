# Hagital-Project-05
Lab: Automate the deployment of a resource group, storage account, and VMs using an Azure Resource Manager (ARM) template.





To automate the deployment of a resource group, storage account, and virtual machines (VMs) in Azure, you can use an Azure Resource Manager (ARM) template. This template will specify the resources you want to deploy in a JSON format, allowing you to deploy them all at once.

Here’s a step-by-step guide to creating an ARM template that deploys a resource group, a storage account, and virtual machines.

Step 1: Define the ARM Template
Create a new JSON file named azuredeploy.json.
Specify the schema and content of the ARM template. Below is an example template that deploys:
A resource group
A storage account
One or more VMs
Here's a sample ARM template:

json
Copy code
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Unique name of the storage account."
      }
    },
    "vmCount": {
      "type": "int",
      "defaultValue": 1,
      "metadata": {
        "description": "Number of VMs to deploy."
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Username for the VM."
      }
    },
    "adminPassword": {
      "type": "secureString",
      "metadata": {
        "description": "Password for the VM."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2022-05-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {}
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2023-07-01",
      "name": "[concat('myVM', copyIndex())]",
      "location": "[parameters('location')]",
      "copy": {
        "name": "vmCopy",
        "count": "[parameters('vmCount')]"
      },
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_DS1_v2"
        },
        "osProfile": {
          "computerName": "[concat('myVM', copyIndex())]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2019-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Standard_LRS"
            }
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', concat('nic', copyIndex()))]"
            }
          ]
        }
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2023-07-01",
      "name": "[concat('nic', copyIndex())]",
      "location": "[parameters('location')]",
      "copy": {
        "name": "nicCopy",
        "count": "[parameters('vmCount')]"
      },
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myVnet', 'default')]"
              },
              "privateIPAllocationMethod": "Dynamic"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-07-01",
      "name": "myVnet",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "default",
            "properties": {
              "addressPrefix": "10.0.0.0/24"
            }
          }
        ]
      }
    }
  ]
}
Step 2: Deploy the ARM Template
Save the template file (azuredeploy.json) to your local machine.

Use the Azure CLI to deploy the template:

bash
Copy code
az deployment group create \
  --resource-group <your-resource-group> \
  --template-file azuredeploy.json \
  --parameters storageAccountName=<unique_storage_account_name> \
              adminUsername=<admin_username> \
              adminPassword=<admin_password>
Replace <your-resource-group> with the name of the resource group.
Replace <unique_storage_account_name> with a globally unique name for the storage account.
Replace <admin_username> and <admin_password> with the desired credentials for the VMs.
Explanation of the Template
Parameters: Define customizable properties like location, storage account name, number of VMs, and admin credentials.
Resources: Define each resource (storage account, VMs, network components).
Copy Loops: Use copy to create multiple instances of a resource (e.g., VMs and NICs).
This template provides a basic setup for deploying a resource group, storage account, and one or more VMs in Azure. You can expand it by adding more parameters or resources as needed.




Step by step to Automate the deployment of a resource group, storage account, and VMs using an Azure Resource Manager (ARM) template.


