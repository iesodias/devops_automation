# 🧪 Lab 0.1 – Introdução à Azure CLI

## 🎯 Objetivo

Aprender os primeiros comandos da Azure CLI para autenticação e consulta prática de recursos (mesmo que ainda não existam recursos provisionados).

---

## 🛠️ Pré-requisitos

* Ter uma conta na Azure
* Ter a Azure CLI instalada ([link oficial](https://learn.microsoft.com/cli/azure/install-azure-cli))

---

## 🚀 Passo a Passo

### 🔐 Passo 1: Fazer login na Azure

```bash
az login
```

### 📋 Passo 2: Listar subscriptions disponíveis

```bash
az account list --output table
```

* Mostra as subscriptions que sua conta tem acesso.
* `--output table` deixa o resultado mais legível.

---

### 📦 Passo 3: Listar Resource Groups existentes

```bash
az group list --output table
```

* Mostra todos os Resource Groups da subscription atual.
* No seu caso, devem aparecer grupos como:

  * `terraform-demo-rg`
  * `cloud-shell-storage-eastus`
  * `NetworkWatcherRG`
  * `DefaultResourceGroup-EUS`
  * `rg-checkov-lab`
  * `rg-tfstate`

> **Atenção**: o comando `az group list` **não aceita o parâmetro `--resource-group`**. Para consultar um grupo específico, use `az group show`.

Exemplo:

```bash
az group show --name terraform-demo-rg --output table
```

---

### 🔍 Passo 4: Listar recursos de um grupo existente (mesmo que esteja vazio)

```bash
az resource list --resource-group terraform-demo-rg --output table
```

* Mostra os recursos dentro do grupo `terraform-demo-rg`.
* Se não houver recursos ainda, o retorno será uma lista vazia.

Você também pode testar com outros grupos existentes:

```bash
az resource list --resource-group rg-checkov-lab --output table
az resource list --resource-group rg-tfstate --output table
```

---

### 📁 Passo 5: Testar listagem de VMs (mesmo que não existam)

```bash
az vm list --output table
```

* Mesmo sem VMs criadas, esse comando confirma que a CLI está funcionando corretamente.

> Exemplo com filtro:

```bash
az vm list --query '[].{name:name, location:location}' --output table
```

---

## ✅ Conclusão

Agora você está autenticado, visualizou suas subscriptions e resource groups reais, e testou comandos para consultar recursos e VMs. Mesmo sem infraestrutura provisionada, você já tem uma base sólida para seguir nos próximos labs. Bora pra prática! 💪

# 🧪 Lab 2 – Criar VMs em lote via Bash + Azure CLI (Baby Steps)

## 🎯 Objetivo

Criar múltiplas máquinas virtuais na Azure primeiro usando comandos manuais e depois automatizar com Bash + Azure CLI.

---

## 🛠️ Pré-requisitos

* Azure CLI instalada
* Já autenticado via `az login`
* Um Resource Group existente (ex: `terraform-demo-rg`)

---

## 🚶 Etapa 1 – Comandos manuais para criar VMs

### 📌 Definir variáveis no terminal:

```bash
az group create --name devops-automation --location eastus
```

```bash
RG="devops-automation"
LOCATION="eastus"
PASSWORD="SenhaForte123"
```

### 💻 Criar a primeira VM manualmente:

```bash
az vm create \
  --resource-group $RG \
  --name vm01 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --admin-password $PASSWORD \
  --authentication-type password \
  --location $LOCATION
```

### 💻 Criar a segunda VM:

```bash
az vm create \
  --resource-group $RG \
  --name vm02 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --admin-password $PASSWORD \
  --authentication-type password \
  --location $LOCATION
```

### 📄 Verificar as VMs criadas:

```bash
az vm list --resource-group $RG --output table
```

---

## 🔍 Etapa 1.1 – Explorar propriedades das VMs criadas

### 1️⃣ Listar IPs públicos

```bash
az vm list-ip-addresses --resource-group $RG --output table
```

### 2️⃣ Ver detalhes da VM `vm01`

```bash
az vm show --resource-group $RG --name vm01 --output json
```

### 3️⃣ Obter apenas o status de `vm01`

```bash
az vm get-instance-view --resource-group $RG --name vm01 --query "instanceView.statuses[*].displayStatus" --output table
```

### 4️⃣ Ver informações do disco da `vm01`

```bash
az disk list --resource-group $RG --output table
```

### 5️⃣ Ver as NICs associadas às VMs

```bash
az network nic list --resource-group $RG --output table
```

### 6️⃣ Ver o grupo de segurança de rede (NSG)

```bash
az network nsg list --resource-group $RG --output table
```

### 7️⃣ Obter IP público diretamente (exemplo com `jq` se tiver instalado)

```bash
az vm list-ip-addresses --resource-group $RG --name vm01 --query "[].virtualMachine.network.publicIpAddresses[].ipAddress" --output tsv
```

---

## 🗑️ Etapa 1.2 – Apagar as VMs antes de recriar

Antes de seguir para a automação, vamos remover as VMs que criamos manualmente:

### 🧼 Comando para apagar `vm01`:

```bash
az vm delete --resource-group $RG --name vm01 --yes
```

### 🧼 Comando para apagar `vm02`:

```bash
az vm delete --resource-group $RG --name vm02 --yes
```

Você pode verificar se as VMs foram removidas com:

```bash
az vm list --resource-group $RG --output table
```

---

## 🤖 Etapa 2 – Transformar em script automatizado

### 📂 Criar estrutura e script:

```bash
mkdir -p ~/labs/azure/vm-lote
cd ~/labs/azure/vm-lote
touch criar-vms.sh
chmod +x criar-vms.sh
```

### ✍️ Conteúdo do `criar-vms.sh`:

```bash
#!/bin/bash

RESOURCE_GROUP="devops-automation"
LOCATION="brazilsouth"
PASSWORD="SenhaForte123!"

criar_vm() {
  local NOME_VM=$1
  echo "Criando VM: $NOME_VM"

  az vm create \
    --resource-group $RESOURCE_GROUP \
    --name $NOME_VM \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --admin-password $PASSWORD \
    --authentication-type password \
    --location $LOCATION \
    --no-wait
}

# Lista de VMs para criar
VMS=(vm01 vm02)

for nome in "${VMS[@]}"; do
  criar_vm $nome
done

echo "Criação em lote iniciada. Aguarde as VMs serem provisionadas."
```

### ▶️ Executar o script:

```bash
./criar-vms.sh
```

# 🧪 Lab 3 – Filtrar e salvar informações com Azure CLI + jq/awk

## 🎯 Objetivo

Aprender a usar comandos de linha de comando para filtrar e salvar dados úteis das VMs — como nome, IP e status — simulando o que faríamos em pipelines como GitHub Actions (mas aqui totalmente via terminal).

---

## 🛠️ Pré-requisitos

* VMs já criadas (ex: `vm01` e `vm02`)
* Azure CLI instalada
* Ferramentas como `jq` e `awk` disponíveis no terminal

---

## 🚶 Etapa 1 – Listar e salvar IPs públicos

### 🧾 Como é a saída bruta antes de filtrar

Se você executar o comando abaixo:

```bash
az vm list-ip-addresses --resource-group devops-automation --output json
```

A saída será semelhante a:

```json
[
  {
    "virtualMachine": {
      "name": "vm01",
      "network": {
        "publicIpAddresses": [
          {
            "ipAddress": "20.120.45.101"
          }
        ]
      }
    }
  },
  {
    "virtualMachine": {
      "name": "vm02",
      "network": {
        "publicIpAddresses": [
          {
            "ipAddress": "20.120.45.102"
          }
        ]
      }
    }
  }
]
```

Essa estrutura será filtrada com `jq` no próximo passo.

### 🔍 Obter IPs com Azure CLI + jq:

Esse comando filtra apenas o nome da VM e o IP público da estrutura JSON anterior, deixando a saída enxuta e fácil de manipular com outras ferramentas.

```bash
az vm list-ip-addresses --resource-group devops-automation \
  --query "[].virtualMachine.{name:name, ip:network.publicIpAddresses[0].ipAddress}" \
  -o json | jq -r '.[] | "\(.name) \(.ip)"'
```

### 💾 Salvar os IPs em um arquivo:

Aqui estamos redirecionando a saída filtrada para o arquivo `vms-ips.txt`, o que é útil para reutilizar essas informações depois em scripts ou etapas futuras de automação.

```bash
az vm list-ip-addresses --resource-group devops-automation \
  --query "[].virtualMachine.{name:name, ip:network.publicIpAddresses[0].ipAddress}" \
  -o json | jq -r '.[] | "\(.name) \(.ip)"' > vms-ips.txt
```

### 📂 Verificar o conteúdo salvo:

Esse comando mostra o conteúdo do arquivo `vms-ips.txt` gerado no passo anterior. Assim conseguimos confirmar que a filtragem e o redirecionamento funcionaram corretamente.

```bash
cat vms-ips.txt
```

```bash
IP_VM1=$(awk '$1 == "vm01" { print $2 }' vms-ips.txt)
echo $IP_VM1

IP_VM2=$(awk '$1 == "vm02" { print $2 }' vms-ips.txt)
echo $IP_VM1
```

---

## 📌 Etapa 2 – Extrair status das VMs

### 🧾 Como é a saída bruta antes de filtrar

Se você executar o comando abaixo para uma VM:

```bash
az vm get-instance-view --resource-group devops-automation --name vm01 --output json
```

Você verá algo parecido com isso dentro da chave `instanceView`:

```json
{
  "statuses": [
    {
      "code": "ProvisioningState/succeeded",
      "displayStatus": "Provisioning succeeded",
      ...
    },
    {
      "code": "PowerState/running",
      "displayStatus": "VM running",
      ...
    }
  ]
}
```

Com base nisso, vamos usar `--query` para extrair apenas o status da energia (ligado/desligado) de forma limpa.

### 🔍 Usar loop para pegar o status atual:

Nesse exemplo, percorremos as VMs `vm01` e `vm02` para coletar seus status atuais (ligada/desligada) usando a Azure CLI e salvamos tudo no arquivo `status.txt`.

```bash
for vm in vm01 vm02; do
  az vm get-instance-view \
    --resource-group devops-automation \
    --name $vm \
    --query "instanceView.statuses[?starts_with(code,'PowerState/')].displayStatus" \
    --output tsv >> status.txt
done
```

### 📂 Exibir os status coletados:

Esse comando imprime o conteúdo de `status.txt`, permitindo verificar o status de execução de cada VM consultada no loop anterior.

```bash
cat status.txt
```

---

## 📋 Etapa 3 – Usar `awk` para reformatar saída

### 🛠️ Exemplo de uso simples:

Esse comando lê o arquivo `vms-ips.txt` e utiliza o `awk` para formatar de forma mais legível, útil quando queremos imprimir logs organizados ou preparar conteúdo para relatório ou integração.

```bash
awk '{ print "VM:" $1 " | IP:" $2 }' vms-ips.txt
```
