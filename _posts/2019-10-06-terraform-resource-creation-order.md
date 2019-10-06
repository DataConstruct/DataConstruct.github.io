---
layout: post
title:  "Terraform Resource Creation Order"
splash: "/assets/img/order.jpg"
excerpt_separator: <!--more-->
---
Creating resources in Terraform may present a problem for users familiar with sequential tools like Chef.

Terraform generates a graph of the dependency tree prior to planning the order of changes. This means your Terraform file is a **desired state**, not an official order.

<!--more-->

For example, creating a subnet within a VNET using the name value as shown below will cause issues because Terraform doesn't understand that it needs to create the VNET before the subnet, even though we provide the eventual name of the VNET (`VNET-NAME`).

```terraform

resource "azurerm_virtual_network" "vnet" {
    name = "VNET-NAME"
}

resource "azurerm_subnet" "vnet_subnet" {
    name = "MY-SUBNET"
    virtual_network_name = "VNET-NAME"
}

```

To tell Terraform to create the VNET first, use an output variable from the VNET (`azurerm_virtual_network.vnet.name`).

```terraform

resource "azurerm_virtual_network" "vnet" {
    name = "VNET-NAME"
}

resource "azurerm_subnet" "vnet_subnet" {
    name = "MY-SUBNET"
    virtual_network_name = azurerm_virtual_network.vnet.name
}

```

This works with any output variable from a resource. Lets say both resources use the same resource group.
You can force the order by using the output resource group name from the first Terraform resource and pipe it into the second Terraform resource. The resource group names have no significance in relation to dependency, but Terraform will assume there is a dependency through the usage of the output variables.

```terraform

resource "azurerm_virtual_network" "vnet" {
    name = "VNET-NAME"
    resource_group_name = "RESOURCE-GROUP"
}

resource "azurerm_subnet" "vnet_subnet" {
    name = "MY-SUBNET"
    virtual_network_name = "VNET-NAME"
    resource_group_name = azurerm_virtual_network.vnet.resource_group_name
}

```

### Defining Dependencies

Terraform also allows you to explicitly define dependencies for each resource, using `depends_on`, which accepts an array of resources.

This can be helpful in situations where it may be hard to pass outputs from one resource to another. The example below tells Terraform that the resource `vnet_subnet` depends on `azurerm_virtual_network.vnet`.

```terraform

resource "azurerm_virtual_network" "vnet" {
    name = "VNET-NAME"
}

resource "azurerm_subnet" "vnet_subnet" {
    name = "MY-SUBNET"
    virtual_network_name = azurerm_virtual_network.vnet.name
    depends_on = [azurerm_virtual_network.vnet] 
}

```

### Learn More

For a more in-depth explanation, see [Terraform Resource Dependencies](https://learn.hashicorp.com/terraform/getting-started/dependencies).

##### Edited by [Melissa Kendall](https://www.linkedin.com/in/melissa-kendall-72b35195/)