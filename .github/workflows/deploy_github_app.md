# deploy_github_app

A reusasble workflow that deploys a GitHub App to an Azure App Service in the form of a Docker image. Leverages OIDC to deploy to Azure from GitHub Actions while using Terraform to manage the required infrastructure.


# Usage
Simply add a `deploy_github_app.yml` file similar to the below to the `.github/workflows` directory of a repository.

```yaml
name: Honey Badger
on:
  workflow_dispatch:
  push:
    branches: ['main']
    paths:
      - 'terraform/**'
jobs:     
  Deploy:
    uses: expert-services/reusable-workflows/.github/workflows/deploy_github_app.yml@main
    with:
      app-name: hb
    secrets: inherit
```

## Prerequisites
- A Federated credential for GitHub Actions must be [correctly configured](https://github.com/marketplace/actions/azure-login#configure-a-federated-credential-to-use-oidc-based-authentication)
  - This Azure App Registration must be given permission (e.g., Contributor) at a scope that allows it to create Resource Groups and the required infrastructure resources
- A GitHub App must be created, with its App ID, webhook secret, and private key (`.pem` file) known 
- The following values must be set as **Repository secrets** in the repository that is calling the reusable workflow
     - **CLIENT_ID**: The client ID of the Azure App Registration used to deploy infrastructure
     - **TENANT_ID**: The tenant ID of the Azure App Registration used to deploy infrastructure
     - **SUBSCRIPTION_ID**: The subscription ID of the Azure subscription that infrastructure is to be deployed to
     - **APP_ID**: The GitHub App ID
     - **WEBHOOK_SECRET**: The Webhook secret specified when creating the GitHub App
     - **PRIVATE_KEY**: The Base64 string associated with the GitHub Apps Private key `.pem` file
 - Terraform files as mentioned in the [Examples](#Examples) sectionmust exists in the `/terraform` directory


## Inputs
| Input     | Required | Description                                                                                |
|-----------|----------|--------------------------------------------------------------------------------------------|
| app-name  | true     | A name that is used to provision the required Terraform state management resources. |
| cloud-provider | false     | The Cloud Service Provider that hosts the required infrastructure for a GitHub App. Must be either `az` or `aws`. |


> **Note**
> Currently only Azure is supported as a Cloud Service Provider, and only workflows that specify `az` as the `cloud-provider` will deploy the GitHub App

# Examples
The below examples demonstrate the capabilities of this reusable workflow.

## Deploying to Azure 
In this example, it is shown how a vulnerable transitive dependency is identified by executing the `workflow_dispatch` trigger. To demonstrate this, [a fork](https://github.com/oodles-noodles/spark) of the apache/spark repository was created.

### Terraform Files
Example `main.tf` file content

```terraform
# The following variables are expected to be present as environment variables 
variable "org" {}
variable "webhook_secret" {}
variable "private_key" {}
variable "app_id" {}
variable "client_id" {}
variable "subscription_id" {}
variable "tenant_id" {}

terraform {
  backend "azurerm" {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "3.64.0"
    }
  }
}

provider "azurerm" {
  use_oidc        = true
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
  client_id       = var.client_id
  features {}
}

resource "azurerm_resource_group" "honeybadger" {
  name     = var.resource-group.name
  location = var.resource-group.location
}

resource "azurerm_service_plan" "honeybadger-asp" {
  name                = var.app-service-plan.name
  location            = azurerm_resource_group.honeybadger.location
  resource_group_name = azurerm_resource_group.honeybadger.name
  os_type             = var.app-service-plan.os-type
  sku_name            = var.app-service-plan.sku-name
}

resource "azurerm_linux_web_app" "honeybadger-app" {
  name = "${var.linux-web-app-name}-${var.org}"
  identity {
    type = "SystemAssigned"
  }

  resource_group_name = azurerm_resource_group.honeybadger.name
  location            = azurerm_service_plan.honeybadger-asp.location
  service_plan_id     = azurerm_service_plan.honeybadger-asp.id
  https_only          = true

  logs {
    http_logs {
      file_system {
        retention_in_days = 4
        retention_in_mb   = 25
      }
    }
    failed_request_tracing = true
  }

  app_settings = {
    "WEBHOOK_SECRET"                    = var.webhook_secret
    "APP_ID"                            = var.app_id
    "PRIVATE_KEY"                       = var.private_key
  }

  site_config {
    application_stack {
      docker_image     = var.docker-config.image
      docker_image_tag = var.docker-config.tag
    }
    http2_enabled                     = true
    ftps_state                        = "Disabled"
    health_check_path                 = "/probot"
    health_check_eviction_time_in_min = 2
  }
}
```

Example `variables.tf` file

```terraform
variable "docker-config" {
  type = object({
    image = string
    tag   = string
  })
  default = {
    image = "githubservices.azurecr.io/honeybadger"
    tag   = "latest"
  }
}

variable "resource-group" {
  type = object({
    name     = string
    location = string
  })
  default = {
    name     = "honeybadger"
    location = "eastus"
  }
}

variable "app-service-plan" {
  type = object({
    name     = string
    os-type  = string
    sku-name = string
  })
  default = {
    name     = "honeybadger-appserviceplan"
    os-type  = "Linux"
    sku-name = "P1v3"
  }
}

variable "linux-web-app-name" {
  type    = string
  default = "honeybadger"
}
```
