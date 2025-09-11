## Lab 2: Monitorando AKS com Datadog (Free Tier)

### Objetivo

Este lab demonstra como criar um cluster AKS no Free Tier e 

---

### 1. Criar estrutura local

```bash
mkdir workspace-aks-gemini
cd workspace-aks-gemini
```

---

### 2. Criar Resource Group no Azure

```bash
az group create --name aks-free-rg --location eastus
```

---

### 3. Criar Cluster AKS (Free Tier)

```bash
az aks create \
  --resource-group aks-free-rg \
  --name aks-free-cluster \
  --node-count 1 \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --enable-managed-identity \
  --tier free \
  --location eastus
```

---

### 4. Configurar acesso ao cluster

```bash
az aks get-credentials --resource-group aks-free-rg --name aks-free-cluster --overwrite-existing
```

---

### 5. Criar manifesto da aplicação

```bash
cat > app-deployment.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-api-deployment
  labels:
    app: java-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: java-api
  template:
    metadata:
      labels:
        app: java-api
    spec:
      containers:
      - name: java-api
        image: iesodias/java-api:latest
        ports:
        - containerPort: 8081
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
---
apiVersion: v1
kind: Service
metadata:
  name: java-api-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8081
    protocol: TCP
  selector:
    app: java-api
EOF
```

---

### 6. Fazer deploy da aplicação

```bash
kubectl apply -f app-deployment.yaml
```


### 7. Validar se a aplicacao subiu


kubectl get service

verificar IP alterar no link abaixo

http://IP//swagger-ui/index.html


---

### 7. Explorar e validar o cluster com Gemini CLI

**Prompt 1 - Aumentar réplicas editando arquivo:**
> "Preciso aumentar o número de réplicas do deployment java-api-deployment de 2 para 3. Você pode editar o arquivo app-deployment.yaml para fazer essa alteração e aplicar a mudança?"

**Prompt 2 - Reduzir réplicas via linha de comando:**
> "Agora preciso voltar o número de réplicas para 2, mas desta vez usando linha de comando diretamente, sem editar arquivo. Você pode fazer isso?"

**Prompt 3 - Validar saúde dos pods:**
> "Você pode verificar a saúde de todos os pods do cluster e me mostrar o status detalhado de cada um?"

**Prompt 4 - Verificar HPA:**
> "Preciso verificar se existe algum HPA (Horizontal Pod Autoscaler) configurado no cluster. Você pode investigar isso?"

**Prompt 5 - Verificar nodes do cluster:**
> "Quantos nodes existem neste cluster AKS? Você pode me mostrar informações detalhadas sobre cada node?"

---

### 8. Simular erro de imagem

```bash
kubectl set image deployment/java-api-deployment java-api=iesodias/jav-api:latest
```

---post mortem

### 9. Troubleshooting com Gemini CLI

**Prompt original:**
> "estou com um problema no meu pod. voce consegue identificar qual é esse problema?"

**Prompt otimizado:**
> "Estou com um problema no meu cluster Kubernetes. Notei que um dos pods está com o status `ImagePullBackOff`. Suspeito que seja um dos pods do deployment `java-api-deployment`. Você pode investigar e corrigir o problema?"

### Por que o prompt otimizado é melhor?

- **Especificidade:** Vai direto ao ponto, mencionando o status do pod (`ImagePullBackOff`)
- **Contexto:** Fornece o nome do deployment (`java-api-deployment`) 
- **Direção:** Pedido claro para "investigar e corrigir"