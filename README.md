Perfect 👍
I’ll give you a clean, interview-ready Terraform script for Azure, aligned with DevOps + Kubernetes + CI/CD use cases.

I’ll cover the most common real-world setup:

> Azure Resource Group + ACR + AKS
(this directly matches GitHub Actions → Docker → Kubernetes pipeline)




---

Architecture (what we are creating)

Terraform
 ├── Resource Group
 ├── Azure Container Registry (ACR)
 └── Azure Kubernetes Service (AKS)

📌 This is 100% resume + interview relevant


---

STEP 1: Prerequisites

Make sure you have:

az login
terraform -version


---

STEP 2: Terraform File Structure

azure-terraform/
├── provider.tf
├── variables.tf
├── main.tf
├── outputs.tf


---

STEP 3: provider.tf

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

📌 Interview line:

> Terraform uses providers to interact with cloud APIs.




---

STEP 4: variables.tf

variable "location" {
  default = "East US"
}

variable "resource_group_name" {
  default = "devops-rg"
}

variable "aks_cluster_name" {
  default = "devops-aks"
}

variable "acr_name" {
  default = "devopsacr12345"
}


---

STEP 5: main.tf (CORE FILE)

1️⃣ Resource Group

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}


---

2️⃣ Azure Container Registry (ACR)

resource "azurerm_container_registry" "acr" {
  name                = var.acr_name
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  sku                 = "Basic"
  admin_enabled       = true
}

📌 Used for storing Docker images.


---

3️⃣ AKS Cluster

resource "azurerm_kubernetes_cluster" "aks" {
  name                = var.aks_cluster_name
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "devopsaks"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_DS2_v2"
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin = "azure"
  }
}

📌 Interview tip:

> AKS uses managed control plane; we manage only worker nodes.




---

4️⃣ Attach ACR to AKS (VERY IMPORTANT)

resource "azurerm_role_assignment" "aks_acr_pull" {
  principal_id         = azurerm_kubernetes_cluster.aks.kubelet_identity[0].object_id
  role_definition_name = "AcrPull"
  scope                = azurerm_container_registry.acr.id
}

📌 This allows AKS to pull images from ACR securely.


---

STEP 6: outputs.tf

output "aks_cluster_name" {
  value = azurerm_kubernetes_cluster.aks.name
}

output "acr_login_server" {
  value = azurerm_container_registry.acr.login_server
}


---

STEP 7: Run Terraform Commands

terraform init
terraform validate
terraform plan
terraform apply

Type yes when asked.


---

STEP 8: Configure kubectl

az aks get-credentials \
  --resource-group devops-rg \
  --name devops-aks

Verify:

kubectl get nodes


---

STEP 9: Push Image to ACR (Manual Test)

az acr login --name devopsacr12345

docker tag my-app devopsacr12345.azurecr.io/my-app:v1
docker push devopsacr12345.azurecr.io/my-app:v1


---

What You Can Say in Interviews 🔥

> “I used Terraform to provision Azure infrastructure including Resource Group, ACR, and AKS.
I also configured RBAC so AKS can securely pull images from ACR.
This setup integrates with GitHub Actions for automated CI/CD deployments.”




---

Want Next?

I can: 1️⃣ Convert this into Terraform modules
2️⃣ Add remote backend (Azure Storage)
3️⃣ Add GitHub Actions pipeline for Terraform
4️⃣ Give Terraform interview Q&A based on this script

Tell me what you want next 🚀