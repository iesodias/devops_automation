# Lab: Introdução ao ArgoCD no AKS

Este laboratório apresenta uma introdução prática ao ArgoCD utilizando o Azure Kubernetes Service (AKS).

## Pré-requisitos

- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) instalado
- [kubectl](https://kubernetes.io/docs/tasks/tools/) instalado
- Conta Azure ativa (pode usar o Free Tier)

---

## Passo 1: Login no Azure

```bash# Lab de Introdução ao ArgoCD no AKS

## O que é GitOps?

**GitOps** é uma metodologia moderna de operações de infraestrutura que utiliza o **Git como única fonte de verdade** para toda a configuração da infraestrutura e aplicações. Em vez de fazer alterações manualmente nos ambientes, todas as mudanças são feitas através de commits no Git.

### Princípios do GitOps

1. **Declarativo**: Todo o estado desejado do sistema é descrito de forma declarativa (YAML, JSON, etc.)
2. **Versionado e Imutável**: Todas as configurações ficam versionadas no Git, permitindo rastreabilidade completa
3. **Pull Automático**: Um agente no cluster monitora o repositório e aplica automaticamente as mudanças
4. **Reconciliação Contínua**: O sistema constantemente verifica se o estado real corresponde ao estado desejado

### Benefícios do GitOps

- **Auditoria Completa**: Todo histórico de mudanças fica registrado no Git
- **Rollback Simples**: Reverter para uma versão anterior é apenas um `git revert`
- **Colaboração**: Uso de Pull Requests para revisão de mudanças
- **Segurança**: Reduz a necessidade de acesso direto aos clusters
- **Automação**: Deploys automáticos quando há mudanças no repositório

---

## O que é ArgoCD?

**ArgoCD** é uma ferramenta declarativa de entrega contínua (CD) para Kubernetes que implementa os princípios do GitOps. Ele monitora repositórios Git e garante que o estado do cluster Kubernetes corresponda ao estado definido no Git.

### Como o ArgoCD Funciona

O ArgoCD opera através de três componentes principais:

1. **Application Controller**: Monitora os repositórios Git e compara o estado desejado com o estado atual do cluster
2. **Repo Server**: Faz o clone dos repositórios Git e gera os manifests Kubernetes
3. **API Server**: Fornece uma interface web e API REST para gerenciamento

### Conceitos Principais

- **Application**: Representa uma aplicação no ArgoCD, vinculando um repositório Git a um namespace no cluster
- **Project**: Agrupa múltiplas aplicações e define políticas de acesso
- **Sync**: Processo de aplicar as mudanças do Git no cluster
- **Health Status**: Indica se os recursos estão saudáveis (Healthy, Degraded, etc.)
- **Sync Status**: Indica se o cluster está sincronizado com o Git (Synced, OutOfSync)

### Tipos de Sincronização

- **Manual**: O usuário precisa clicar em "Sync" na interface
- **Automática**: O ArgoCD aplica automaticamente as mudanças detectadas no Git
- **Self-Heal**: Se alguém modificar recursos diretamente no cluster, o ArgoCD reverte para o estado do Git
- **Prune**: Remove recursos que não existem mais no Git

### Vantagens do ArgoCD

- **Interface Visual**: Dashboard web intuitivo para visualizar aplicações
- **Multi-Cluster**: Gerencia múltiplos clusters Kubernetes de um único lugar
- **Rollback**: Fácil reversão para versões anteriores
- **SSO/RBAC**: Integração com sistemas de autenticação corporativos
- **Hooks**: Execução de scripts antes/depois do deploy
- **Progressive Delivery**: Suporte a Canary e Blue/Green deployments

---

## Sobre Este Lab

Este laboratório foi criado para fornecer uma **introdução prática ao ArgoCD** utilizando o **Azure Kubernetes Service (AKS)** como plataforma. O objetivo é demonstrar, de forma clara e objetiva, como implementar uma pipeline GitOps funcional.

### Objetivo do Lab

Demonstrar na prática:
- Como instalar e configurar o ArgoCD em um cluster Kubernetes
- Como conectar o ArgoCD a um repositório Git
- Como o ArgoCD monitora e aplica mudanças automaticamente
- Como funciona o conceito de sincronização automática
- Como o GitOps simplifica o gerenciamento de aplicações

### O que o Lab Demonstra

**Infraestrutura**
- Criação de um cluster AKS no modo mais econômico (Free Tier)
- Instalação do ArgoCD usando manifests oficiais
- Configuração de acesso via port-forward

**GitOps em Ação**
- Manifests Kubernetes versionados no Git (Deployment + Service)
- Application do ArgoCD apontando para o repositório
- Sincronização automática a cada 3 minutos
- Deploy automático quando há mudanças no Git

**Aplicação de Exemplo**
- Task Manager: aplicação web de gerenciamento de tarefas
- Desenvolvida em Python com FastAPI
- Interface web responsiva + API REST
- Documentação Swagger integrada

### Fluxo do Lab

1. **Preparação**: Criação da infraestrutura no Azure (AKS)
2. **Setup**: Instalação e configuração do ArgoCD
3. **Configuração GitOps**: Criação dos manifests e Application
4. **Versionamento**: Push dos arquivos para o GitHub
5. **Deploy Automático**: ArgoCD detecta e aplica as mudanças
6. **Validação**: Testes da aplicação e endpoints
7. **Demonstração**: Mudança de réplicas para validar sync automático

### Arquitetura do Lab

O lab implementa uma arquitetura GitOps simples mas completa:

**GitHub** → Repositório Git com os manifests da aplicação

↓ (Polling a cada 3 minutos)

**ArgoCD** → Monitora o repositório e detecta mudanças

↓ (Aplica automaticamente)

**Kubernetes (AKS)** → Cluster onde a aplicação roda

↓ (Expõe externamente)

**LoadBalancer** → Acesso público à aplicação

### Diferenciais do Lab

- **Econômico**: Usa tier gratuito do AKS e VMs de baixo custo
- **Completo**: Cobre todo o ciclo GitOps, do setup ao deploy
- **Prático**: Aplicação real funcionando com API REST
- **Educacional**: Explicações claras em cada etapa
- **Realista**: Demonstra o tempo de polling do ArgoCD (3 min)

### Conceitos Aprendidos

Ao completar este lab, você terá compreendido:

- ✅ O que é GitOps e seus benefícios
- ✅ Como o ArgoCD implementa GitOps
- ✅ Como conectar ArgoCD a um repositório Git
- ✅ Como funciona a sincronização automática
- ✅ Como o Self-Heal mantém o estado desejado
- ✅ Como deployar aplicações via GitOps
- ✅ Como escalar aplicações modificando o Git

### Casos de Uso Reais

Esta arquitetura GitOps com ArgoCD é utilizada em ambientes de produção para:

- **Continuous Deployment**: Deploys automáticos quando desenvolvedores fazem merge
- **Multi-Ambientes**: Gerenciar dev, staging e produção do mesmo repositório
- **Disaster Recovery**: Recriar ambientes completos a partir do Git
- **Compliance**: Auditoria completa de todas as mudanças
- **Multi-Cluster**: Propagar configurações para múltiplos clusters

### Próximos Passos

Após dominar este lab básico, explore:

- **Helm Charts**: Deploy de aplicações usando Helm com ArgoCD
- **Kustomize**: Gerenciamento de configurações multi-ambiente
- **ApplicationSets**: Deploy de múltiplas aplicações simultaneamente
- **Progressive Delivery**: Implementação de Canary Deployments
- **Webhooks**: Sincronização instantânea via GitHub webhooks
- **ArgoCD Image Updater**: Atualização automática de tags de imagem
- **Secrets Management**: Integração com Sealed Secrets ou Vault

---

## Conclusão

Este lab demonstra como o **GitOps com ArgoCD** transforma o processo de deploy de aplicações em Kubernetes, tornando-o mais **seguro, auditável e automatizado**. O Git passa a ser a única fonte de verdade, e qualquer mudança é rastreável e reversível.

A combinação de **ArgoCD + Kubernetes + Git** representa o estado da arte em entrega contínua, sendo amplamente adotada por empresas que buscam modernizar suas operações e implementar práticas DevOps maduras.

---

## Recursos Adicionais

### Documentação Oficial
- [ArgoCD Official Docs](https://argo-cd.readthedocs.io/)
- [GitOps Principles (OpenGitOps)](https://opengitops.dev/)
- [Azure AKS Documentation](https://docs.microsoft.com/azure/aks/)

### Comunidade
- [ArgoCD GitHub](https://github.com/argoproj/argo-cd)
- [CNCF ArgoCD Project](https://www.cncf.io/projects/argo/)
- [ArgoCD Slack Channel](https://argoproj.github.io/community/join-slack/)

### Aprendizado
- [ArgoCD Tutorial](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [GitOps with ArgoCD (Coursera)](https://www.coursera.org/courses?query=gitops)
- [Kubernetes Patterns Book](https://k8spatterns.io/)

---

**Autor**: Criado para fins educacionais - Lab de Introdução ao ArgoCD no AKS

**Licença**: Livre para uso educacional e aprendizado

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