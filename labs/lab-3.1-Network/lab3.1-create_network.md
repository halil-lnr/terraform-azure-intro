# Create a Network

Lab Objective:
- Create a simple network in Azure

## Lab

Open the file "main.tf" for edit.  Make the following changes to the file:

1. Add Azure RM as a required provider.  Keep the random provider (we will be using it again in a later lab). We will continue to use Azure for storing backend state.

```
terraform {
  required_providers {
    random = {
      source  = "hashicorp/random"
      version = "~> 2.3.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 2.40, < 3.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "terraform-course-backend"
    container_name       = "tfstate"
    key                  = "cprime.terraform.labs.tfstate"
  }
  required_version = ">= 1.0.0"
}
```

2. Add a provider block to configure the Azure RM provider.  We will be using the default configuration of the provider. *Do not delete the "random" provider.*

   (The "provider" referred to in the "skip_provider_registration" flag is a Microsoft provider configured in the Azure subscription, which is not the same as the Azure RM provider for Terraform.  Sorry for the confusion.)

```
provider "azurerm" {
  features {}
  # Set the following flag to avoid an Azure subscription configuration error
  skip_provider_registration = true
}
```

3. Delete the existing "random_integer" resource from the file.

4. Add a resource for an Azure resource group.  In Azure, most resources need to be part of a resource group, so this resource group will be referenced extensively.

```
resource "azurerm_resource_group" "lab" {
  name     = "aztf-labs-rg"
  location = "westus2"
  tags     = {
    Environment = "Lab"
    Project     = "AZTF Training"
  }
}
```

5. Add a resource for a virtual network.  

```
resource "azurerm_virtual_network" "lab" {
  name                = "aztf-labs-vnet"
  location            = "westus2"
  resource_group_name = azurerm_resource_group.lab.name
  address_space       = ["10.0.0.0/16"]
  tags                = {
    Environment = "Lab"
    Project     = "AZTF Training"
  }
}
```

6. Add a resource for two subnets (public and private subnets).  Notice that the subnet CIDR blocks are within the VNet CIDR range.

```
resource "azurerm_subnet" "lab-public" {
  name                 = "aztf-labs-subnet-public"
  resource_group_name  = azurerm_resource_group.lab.name
  virtual_network_name = azurerm_virtual_network.lab.name
  address_prefixes     = ["10.0.0.0/24"]
}

resource "azurerm_subnet" "lab-private" {
  name                 = "aztf-labs-subnet-private"
  resource_group_name  = azurerm_resource_group.lab.name
  virtual_network_name = azurerm_virtual_network.lab.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

7. Add a resource for a security group.  For now we do not include a security group rule.

```
resource "azurerm_network_security_group" "lab-public" {
  name                = "aztf-labs-public-sg"
  location            = "westus2"
  resource_group_name = azurerm_resource_group.lab.name
}
```

8. Add a resource to associate the security group to the public subnet.

```
resource "azurerm_subnet_network_security_group_association" "lab-public" {
  subnet_id                 = azurerm_subnet.lab-public.id
  network_security_group_id = azurerm_network_security_group.lab-public.id
}
```

Run terraform init.  Do you know why we need to re-run this command?
```
terraform init
```
Run terraform validate.
```
terraform validate
```
Run terraform plan.
```
terraform plan
```
(If you see in the plan that random_integer.number is to be destroyed, that is okay.  Do you know why this is happening?)

![Random integer destroy warning](./images/tf-plan.png "Random integer destroy warning")
<br /><br />

Run terraform apply to create all the new infrastructure.
```
terraform apply
```

### Viewing Results in the Azure Portal

Let's use the Azure Portal to see what we just created.  Minimize the Cloud Shell console so you can see the Azure Portal UI fully.

In the search bar at the top of the Azure Portal page, type in “resource”.  Select “Resource Groups” from the auto suggest drop-down.

You should see the following. (If you do not see the aztf-labs-rg resource group, click the Refresh icon above the resource list.)

![Azure Resource Groups](./images/az-rg.png "Azure Resource Groups")
<br /><br />

Click on the "aztf-labs-rg" resource group created by Terraform. (The other resource groups were created by other means in support of this class. You can ignore them.)

Confirm you see the virtual network and security group listed.<br />

![Resource Group containing virtual network and security group](./images/az-rg-vnet.png "Resource Group containing virtual network and security group")

<br /><br />
Click on the virtual network and confirm it has the expected subnets, and that the public subnet has the expected security group.

![Virtual network subnets and security group](./images/az-vnet-subnets.png "Virtual network subnets and security group")
