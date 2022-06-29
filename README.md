# tp4-terraform


## Initialisation 
Initialiser terraform en précisant le provider et la version

```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.0.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
  subscription_id = "765266c6-9a23-4638-af32-dd1e32613047"
}
```

## Expoloiter les resources, group, virtual network, subnet, public_ip, network interface et la machine virtuelle 

```
 data "azurerm_resource_group" "tp4" {
  name = "devops-TP2"
}

output "id" {
  value = data.azurerm_resource_group.tp4.id
}

 data "azurerm_virtual_network" "example-network" {
  name                = "example-network"
  resource_group_name = "devops-TP2"
}

output "virtual_network_id" {
  value = data.azurerm_virtual_network.example-network.id
}

 data "azurerm_subnet" "test" {
  name                 = "backend"
  virtual_network_name = "example-network"
  resource_group_name  = "devops-TP2"
}

output "subnet_id" {
  value = data.azurerm_subnet.test.id
}

 resource "azurerm_public_ip" "test" {
   name                         = "publicIPForLB"
   location                     = data.azurerm_resource_group.tp4.location
   resource_group_name          =data.azurerm_resource_group.tp4.name
   allocation_method            = "Static"
 }

 resource "azurerm_network_interface" "example" {
  name                = "example-nic"
  location            = data.azurerm_resource_group.tp4.location
  resource_group_name = data.azurerm_resource_group.tp4.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = data.azurerm_subnet.test.id
    private_ip_address_allocation = "Dynamic"
  }
}



 resource "azurerm_linux_virtual_machine" "devops-20210970" {
   count                 = 2
   name                  = "acctvm${count.index}"
   location              = data.azurerm_resource_group.tp4.location
   resource_group_name   = data.azurerm_resource_group.tp4.name
   
    size               = "Standard_D2s_v2"
    admin_username      = "devops"
   network_interface_ids = [
    azurerm_network_interface.example.id,
  ]
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
   source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "16.04-LTS"
    version   = "latest"
  }

   tags = {
     environment = "staging"
   }
 }
```
## Initialisation Azure 
Se connecter sur Azure 
```
$ az login
```
Affecter la souscription azure 
```
$ az account set --subscription="765266c6-9a23-4638-af32-dd1e32613047"
```
## Exécuter les commandes terraform :
pour initialiser terraform
```
$ terraform init
```
Créer le plan d'exécution 

```
$ terraform plan -out main.tfplan
```
Appliquer le plan d'exécution sur l'infrastructure cloud azure 

```
$ terraform apply main.tfplan
```

## Les erreurs : 
```
│ Error: Error: Subnet: (Name "backend" / Virtual Network Name "example-network" / Resource Group "devops-TP2") was not found
│ 
│   with data.azurerm_subnet.test,
│   on terraform.tf line 33, in data "azurerm_subnet" "test":
│   33:  data "azurerm_subnet" "test" {
```
