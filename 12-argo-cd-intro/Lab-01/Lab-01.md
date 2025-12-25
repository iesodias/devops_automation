# Lab: Introdução ao ArgoCD no AKS

Este laboratório apresenta uma introdução prática ao ArgoCD utilizando o Azure Kubernetes Service (AKS).

## Pré-requisitos

- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) instalado
- [kubectl](https://kubernetes.io/docs/tasks/tools/) instalado
- Conta Azure ativa (pode usar o Free Tier)

---

## Passo 1: Login no Azure

```bash
az login
```

## Passo 2: Criar Resource Group

```bash
az group create --name rg-argocd-lab --location eastus
```

## Passo 3: Criar o Cluster AKS (Modo Econômico - Free Tier)

Este comando cria um cluster AKS com a configuração mais econômica possível:

```bash
az aks create \
  --resource-group rg-argocd-lab \
  --name aks-argocd-lab \
  --node-count 1 \
  --node-vm-size Standard_B2s \
  --generate-ssh-keys \
  --tier free
```

> **Nota:** O `Standard_B2s` é uma das VMs mais baratas disponíveis. O tier `free` remove custos do control plane.

## Passo 4: Obter Credenciais do Cluster

```bash
az aks get-credentials --resource-group rg-argocd-lab --name aks-argocd-lab
```

## Passo 5: Verificar Conexão com o Cluster

```bash
kubectl get nodes
```

---

## Passo 6: Instalar o ArgoCD

### 6.1 Criar Namespace para o ArgoCD

```bash
kubectl create namespace argocd
```

### 6.2 Instalar o ArgoCD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 6.3 Aguardar os Pods Ficarem Prontos

```bash
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s
```

### 6.4 Verificar os Pods do ArgoCD

```bash
kubectl get pods -n argocd
```

---

## Passo 7: Acessar o ArgoCD

### 7.1 Expor o ArgoCD Server (Port Forward)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

> Acesse em: https://localhost:8080

### 7.2 Obter a Senha Inicial do Admin

Em outro terminal:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

> **Credenciais:**
> - **Usuário:** admin
> - **Senha:** (resultado do comando acima)

---

## Passo 8: Criar Manifests da Aplicação

Crie a estrutura de diretórios para os manifests:

```bash
mkdir -p manifests
```

### 8.1 Criar o Manifesto da Aplicação (`manifests/app.yaml`)

Este arquivo contém o Deployment e o Service em um único arquivo:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: task-manager
  labels:
    app: task-manager
spec:
  replicas: 2
  selector:
    matchLabels:
      app: task-manager
  template:
    metadata:
      labels:
        app: task-manager
    spec:
      containers:
      - name: task-manager
        image: iesodias/task-manager:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "250m"
---
apiVersion: v1
kind: Service
metadata:
  name: task-manager-service
spec:
  selector:
    app: task-manager
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
```

---

## Passo 9: Criar o Application do ArgoCD (`application.yaml`)

Crie o arquivo `application.yaml` na **raiz do projeto** (fora da pasta manifests):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: task-manager
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/iesodias/aks_argo.git
    targetRevision: main
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

> **Nota:** O ArgoCD monitora o repositório Git e aplica automaticamente as mudanças.

---

## Passo 10: Enviar os Manifests para o GitHub

O ArgoCD é uma ferramenta **GitOps**, ou seja, ele busca os manifests de um repositório Git. Por isso, precisamos enviar os arquivos para o GitHub.

### 10.1 Inicializar o Repositório Git

```bash
git init
git add .
git commit -m "Add ArgoCD lab files"
```

### 10.2 Criar o Repositório no GitHub

Acesse https://github.com/new e crie um repositório chamado `aks_argo`.

### 10.3 Enviar para o GitHub

```bash
git remote add origin https://github.com/iesodias/aks_argo.git
git branch -M main
git push -u origin main
```

---

## Passo 11: Aplicar o Application no ArgoCD

```bash
kubectl apply -f application.yaml
```

O ArgoCD irá automaticamente sincronizar e fazer o deploy da aplicação a partir do repositório Git.

---

## Passo 12: Verificar Deploy

### 12.1 Verificar Application no ArgoCD

```bash
kubectl get application -n argocd
```

O status deve mostrar **Synced** e **Healthy**.

### 12.2 Verificar Pods

```bash
kubectl get pods -l app=task-manager
```

### 12.3 Verificar Service

```bash
kubectl get svc task-manager-service
```

### 12.4 Obter IP Externo

```bash
kubectl get svc task-manager-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

---

## Passo 13: Testar a Aplicação

Após obter o IP externo do LoadBalancer, substitua `<EXTERNAL-IP>` pelo IP obtido no passo anterior.

### 13.1 Testar Página Inicial

```bash
curl http://<EXTERNAL-IP>/
```

### 13.2 Testar Health Check

```bash
curl http://<EXTERNAL-IP>/api/health
```

### 13.3 Listar Tarefas

```bash
curl http://<EXTERNAL-IP>/api/tasks
```

### 13.4 Criar uma Tarefa

```bash
curl -X POST http://<EXTERNAL-IP>/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Minha primeira tarefa","description":"Teste do ArgoCD","priority":"high"}'
```

### 13.5 Ver Estatísticas

```bash
curl http://<EXTERNAL-IP>/api/stats
```

### 13.6 Acessar Swagger UI

Acesse no navegador:

```
http://<EXTERNAL-IP>/docs
```

---

## Endpoints Disponíveis

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/` | Página inicial (HTML) |
| GET | `/docs` | Swagger UI (Documentação interativa) |
| GET | `/api/health` | Health check da aplicação |
| GET | `/api/tasks` | Listar todas as tarefas |
| POST | `/api/tasks` | Criar nova tarefa |
| GET | `/api/tasks/{id}` | Obter tarefa por ID |
| PUT | `/api/tasks/{id}` | Atualizar tarefa |
| DELETE | `/api/tasks/{id}` | Deletar tarefa |
| GET | `/api/stats` | Estatísticas das tarefas |

---

## Limpeza do Ambiente

### Deletar a Aplicação do ArgoCD

```bash
kubectl delete -f application.yaml
```

### Deletar o Cluster AKS

```bash
az aks delete --resource-group rg-argocd-lab --name aks-argocd-lab --yes --no-wait
```

### Deletar o Resource Group

```bash
az group delete --name rg-argocd-lab --yes --no-wait
```

---

## Arquitetura do Lab

```
┌─────────────────────────────────────────────────────────┐
│                     GitHub                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Repository: iesodias/aks_argo                     │  │
│  │  └── manifests/app.yaml                            │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                            │
                            │ git pull (GitOps)
                            ▼
┌─────────────────────────────────────────────────────────┐
│                      Azure Cloud                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │                   AKS Cluster                      │  │
│  │  ┌─────────────────┐  ┌─────────────────────────┐ │  │
│  │  │    ArgoCD       │  │    Default Namespace    │ │  │
│  │  │  ┌───────────┐  │  │  ┌───────────────────┐  │ │  │
│  │  │  │  Server   │  │  │  │   Task Manager    │  │ │  │
│  │  │  │  Repo     │──┼──┼─▶│   Deployment      │  │ │  │
│  │  │  │  Controller│ │  │  │   (2 replicas)    │  │ │  │
│  │  │  └───────────┘  │  │  └───────────────────┘  │ │  │
│  │  └─────────────────┘  │  ┌───────────────────┐  │ │  │
│  │                       │  │  LoadBalancer     │  │ │  │
│  │                       │  │  Service          │  │ │  │
│  │                       │  └───────────────────┘  │ │  │
│  │                       └─────────────────────────┘ │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

**Estrutura de Arquivos:**

```
aks_argo/
├── manifests/
│   └── app.yaml          # Deployment + Service
├── application.yaml      # ArgoCD Application
└── README.md
```

---

## Referências

- [Documentação Oficial do ArgoCD](https://argo-cd.readthedocs.io/)
- [Azure AKS Documentation](https://docs.microsoft.com/azure/aks/)
- [ArgoCD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)

---