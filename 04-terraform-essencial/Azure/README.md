# Lab 1 – Criando um Resource Group com Terraform (Azure)

## 🎯 Objetivo
Criar um Resource Group na Azure usando Terraform, passo a passo.

## 📁 Estrutura de Diretórios
```bash
mkdir -p ~/labs/terraform/lab1-resource-group
cd ~/labs/terraform/lab1-resource-group
```

## 📄 Passo 1 – Criar o arquivo de provider
```bash
export ARM_SUBSCRIPTION_ID="SUA_SUBSCRIPTION_AQUI"
vi provider.tf
```
Conteúdo:
```t
provider "azurerm" {
  features {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.0"
}
```

## 📄 Passo 2 – Criar o arquivo de configuração principal
```bash
vi main.tf
```
Conteúdo:
```hcl
resource "azurerm_resource_group" "rg" {
  name     = "devops-rg"
  location = "East US"
}
```

## ✅ Passo 3 – Inicializar o Terraform
```bash
terraform init
```

## 🔍 Passo 4 – Validar e revisar o plano
```bash
terraform validate
terraform plan
```

## 🚀 Passo 5 – Aplicar o código e provisionar
```bash
terraform apply
```
Confirme com `yes` quando solicitado.

## 🧼 Passo 6 – Destruir os recursos (opcional)
```bash
terraform destroy
```

---

# Lab 2 – Criando uma VM Ubuntu com Variáveis e Data Source (Azure)

## 🎯 Objetivo
Criar uma máquina virtual Ubuntu na Azure usando Terraform com variáveis e buscando o resource group existente via `data`.

## 📁 Estrutura de Diretórios
```bash
mkdir -p ~/labs/terraform/lab2-vm-variaveis
cd ~/labs/terraform/lab2-vm-variaveis
```

## 📄 Passo 1 – Criar o arquivo de provider
```bash
vi provider.tf
```
Conteúdo:
```t
provider "azurerm" {
  features {}
}

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
  }
  required_version = ">= 1.0"
}
```

## 📄 Passo 2 – Definir variáveis
```bash
vi variables.tf
```

Conteúdo:
```t
variable "location" {
  default = "East US"
}

variable "resource_group_name" {
  default = "devops-rg"
}

variable "admin_username" {
  default = "azureuser"
}

variable "admin_password" {
  default = "SenhaForte123!@#"
}
```

## 📄 Passo 3 – Criar o arquivo de configuração principal
```bash
vi main.tf
```
Conteúdo:
```t
data "azurerm_resource_group" "rg" {
  name = var.resource_group_name
}

resource "azurerm_public_ip" "public_ip" {
  name                = "devops-public-ip"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Basic"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "devops-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name
}

resource "azurerm_subnet" "subnet" {
  name                 = "devops-subnet"
  resource_group_name  = data.azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_network_interface" "nic" {
  name                = "devops-nic"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.public_ip.id
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "devops-vm"
  resource_group_name = data.azurerm_resource_group.rg.name
  location            = var.location
  size                = "Standard_B1s"
  admin_username      = var.admin_username
  admin_password      = var.admin_password
  disable_password_authentication = false
  network_interface_ids = [
    azurerm_network_interface.nic.id
  ]
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts"
    version   = "latest"
  }
}

resource "azurerm_network_security_group" "nsg" {
  name                = "devops-nsg"
  location            = var.location
  resource_group_name = data.azurerm_resource_group.rg.name

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "nic_nsg" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}
```

## ✅ Passo 4 – Inicializar o Terraform
```bash
terraform init
```

## 🔍 Passo 5 – Validar e revisar o plano
```bash
terraform validate
terraform plan
```

## 🚀 Passo 6 – Aplicar o código e provisionar
```bash
terraform apply
```
Confirme com `yes` quando solicitado.

## 🧼 Passo 7 – Destruir os recursos (opcional)
```bash
terraform destroy
```

---

# Lab 01 – Criando um AKS com Terraform (Camada Gratuita)

## O Que Vamos Fazer

- Criar um cluster AKS usando Terraform
- Usar a camada gratuita do Azure (Free Tier)
- Autenticar via Azure CLI
- Aplicar a infraestrutura com comandos passo a passo
- Validar e acessar o cluster com `kubectl`

---

## Estrutura de Diretórios

```bash
aks-free-tier/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── keda.tf
├── helm-ingress.tf
```

```bash
mkdir -p ~/labs/terraform/aks-free-tier
cd ~/labs/terraform/aks-free-tier

touch main.tf variables.tf outputs.tf provider.tf keda.tf helm-ingress.tf
```

---

1. Use the curl command to download the kubectl binary:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
This command fetches the latest stable version of kubectl and downloads the appropriate binary for the AMD64 architecture.

Give execute permission to the binary:
```bash
chmod +x kubectl
```
Move the kubectl binary to a directory in your PATH. For example, you can move it to /usr/local/bin/:
```bash
sudo mv kubectl /usr/local/bin/
```
Verify that the installation was successful by running the following command:
```bash
kubectl version --client
```

### Download the Helm installation script:

1. Download a file
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

2. Make the script executable:

```bash
chmod +x get_helm.sh
```
3. Run the installation script to install Helm:

```bash
./get_helm.sh
```
4. Verify the installation by checking the Helm version:

```bash
helm version
```

### How to install AzCli

```bash
sudo apt install azure-cli
```

### Atualizar pacotes
```bash
sudo apt update && sudo apt upgrade -y
```
###  Instalar dependências
```bash
sudo apt install -y gnupg software-properties-common curl
```

###  Adicionar chave GPG da HashiCorp
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
```

###  Adicionar repositório oficial
```bash
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```

###  Atualizar repositórios e instalar Terraform
```bash
sudo apt update
sudo apt install -y terraform
```

# Verificar instalação
```bash
terraform -v
```

## provider.tf

```t
provider "azurerm" {
  features {}
}

terraform {
  required_version = ">= 1.0.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    kubernetes = {
      source = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
    helm = {
      source = "hashicorp/helm"
      version = "~> 2.0"
    }
  }
}

provider "kubernetes" {
  host                   = azurerm_kubernetes_cluster.aks.kube_config[0].host
  client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_certificate)
  client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_key)
  cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].cluster_ca_certificate)
}

provider "helm" {
  kubernetes {
    host                   = azurerm_kubernetes_cluster.aks.kube_config[0].host
    client_certificate     = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_certificate)
    client_key             = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].client_key)
    cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.aks.kube_config[0].cluster_ca_certificate)
  }
}
```

---

## variables.tf

```hcl
variable "resource_group_name" {
  default = "rg-aks-free"
}

variable "location" {
  default = "eastus"
}

variable "aks_cluster_name" {
  default = "aks-free-tier"
}
```

---

## main.tf

```t
data "azurerm_kubernetes_cluster" "aks_data" {
  name                = azurerm_kubernetes_cluster.aks.name
  resource_group_name = azurerm_kubernetes_cluster.aks.resource_group_name
}

resource "azurerm_resource_group" "rg" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_kubernetes_cluster" "aks" {
  name                = var.aks_cluster_name
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "aksfree"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_B2s"  # ideal para camada gratuita
  }

  identity {
    type = "SystemAssigned"
  }

  tags = {
    Environment = "Dev"
  }
}

resource "azurerm_public_ip" "ingress_ip" {
  name                = "myAKSPublicIPForIngress"
  location            = azurerm_resource_group.rg.location
  resource_group_name = data.azurerm_kubernetes_cluster.aks_data.node_resource_group
  allocation_method   = "Static"
  sku                 = "Standard"
}
```

---


## helm_ingress.tf

```t
resource "kubernetes_namespace" "ingress" {
  metadata {
    name = "ingress-nginx"
  }
}

resource "helm_release" "nginx_ingress" {
  name       = "ingress-nginx"
  namespace  = kubernetes_namespace.ingress.metadata[0].name
  repository = "https://kubernetes.github.io/ingress-nginx"
  chart      = "ingress-nginx"
  version    = "4.9.1"

  values = [
    <<EOF
controller:
  replicaCount: 2
  nodeSelector:
    beta.kubernetes.io/os: linux
  service:
    type: LoadBalancer
    externalTrafficPolicy: Local
    loadBalancerIP: "${azurerm_public_ip.ingress_ip.ip_address}"

defaultBackend:
  nodeSelector:
    beta.kubernetes.io/os: linux
EOF
  ]

  depends_on = [azurerm_public_ip.ingress_ip]
}

```

## helm_keda.tf

```t
resource "kubernetes_namespace" "keda" {
  metadata {
    name = "keda"
  }
}

resource "helm_release" "keda" {
  name       = "keda"
  namespace  = kubernetes_namespace.keda.metadata[0].name
  repository = "https://kedacore.github.io/charts"
  chart      = "keda"
  version    = "2.13.2"

  values = [
    <<EOF
replicaCount: 1
EOF
  ]

  depends_on = [kubernetes_namespace.keda]
}

```

## outputs.tf

```t

output "next_steps" {
  value = <<EOT
🎉 Cluster AKS criado com sucesso!

👉 Para acessar seu cluster, execute o seguinte comando:

az aks get-credentials --resource-group ${var.resource_group_name} --name ${var.aks_cluster_name}

🚀 Ingress Controller instalado com sucesso!

✅ Acesse o IP fixo criado:
http://${azurerm_public_ip.ingress_ip.ip_address}

📄 Você verá uma mensagem "404 Not Found" do NGINX — isso é esperado!

➡️ Prossiga para o Lab 03 para criar uma aplicação + Ingress Resource.

EOT
}

output "resource_group_name" {
  value = var.resource_group_name
}

output "aks_cluster_name" {
  value = var.aks_cluster_name
}
```

---

## Passo a Passo para Rodar

### Pré-requisitos

- Instalar [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- Instalar [Terraform](https://developer.hashicorp.com/terraform/downloads)
- Ter uma conta Azure com créditos ou Free Tier

---

### 1. Login no Azure

```bash
az login
```

---

### 2. Inicializar o Terraform

```bash
terraform init
```

---

### 3. Visualizar o plano de execução

```bash
terraform plan
```

---

### 4. Aplicar a infraestrutura

```bash
terraform apply -auto-approve
```

---

### 5. Acessar o cluster AKS

```bash
az aks get-credentials --resource-group rg-aks-free --name aks-free-tier
kubectl get nodes
```

# Lab 2 - Criando seu primeiro Pod no Kubernetes (porta 8081)

## Objetivo

Criar manualmente um Pod com a imagem `iesodias/java-api:latest`, rodando na porta `8081`, e interagir com ele via `kubectl`.

---
