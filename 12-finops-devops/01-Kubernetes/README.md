## Lab 1: FinOps no Kubernetes (AKS) — Right-Sizing de Pods

### Objetivo

Neste lab você vai aprender, na prática, uma das principais práticas de **FinOps em Kubernetes**: o **right-sizing** (dimensionamento correto) de pods.

O erro mais comum (e mais caro) em produção é o **superdimensionamento**: por medo de instabilidade, as pessoas colocam `requests` e `limits` muito **maiores** do que a aplicação realmente usa.

O problema é que no Kubernetes o **`requests` reserva recurso no node**. Mesmo que o pod use quase nada de CPU/memória, aquele recurso fica **bloqueado** para os outros. Resultado:

- Os nodes "enchem" no papel (reserva), mas ficam **ociosos** no uso real.
- Você precisa de **mais nodes** (ou nodes maiores) → **paga mais** sem necessidade.
- Pods novos podem ficar **`Pending`** por "falta" de recurso que na verdade está sobrando.

O objetivo do FinOps é achar o **equilíbrio**: reservar **exatamente o que a aplicação precisa** (consumo real + uma folga), nem mais nem menos.

Para isso vamos:

1. Provisionar um cluster AKS simples com `az cli`.
2. Subir a aplicação **propositalmente superdimensionada**.
3. Fazer o **troubleshooting** e provar que estamos **reservando muito e usando pouco** (desperdício de dinheiro).
4. **Ajustar (right-sizing para baixo)** os recursos e validar a economia.

> 💡 **Sobre custos:** o lab usa um único node pequeno e, ao final, **destruímos tudo** para não gerar custo.

**Aplicação usada no lab:** `iesodias/java-api:latest` (API, expõe a porta `8080`).

---

### Pré-requisitos

- `az cli` instalado e atualizado → [Instalar o Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)
- `kubectl` instalado → o `az aks install-cli` instala para você
- Uma subscription Azure ativa

```bash
# Verificar versões
az version
kubectl version --client
```

---

### 1. Criar estrutura local

```bash
mkdir workspace-finops-aks
cd workspace-finops-aks
```

---

### 2. Login no Azure e variáveis

```bash
az login

# Variáveis do lab (ajuste se quiser)
export RG="finops-aks-rg"
export CLUSTER="finops-aks-cluster"
export LOCATION="eastus"
```

> Se tiver mais de uma subscription, selecione a correta:
> ```bash
> az account set --subscription "<NOME_OU_ID_DA_SUBSCRIPTION>"
> ```

---

### 3. Criar o Resource Group

```bash
az group create --name $RG --location $LOCATION
```

---

### 4. Criar o Cluster AKS (simples e barato)

Usamos **1 node** com VM pequena (`Standard_B2s` = **2 vCPU / 4 GB**) e o **tier Free** do control plane. Esse tamanho é importante: ele torna o desperdício **visível**, porque a reserva alta dos pods vai "encher" o node rápido.

```bash
az aks create \
  --resource-group $RG \
  --name $CLUSTER \
  --location $LOCATION \
  --node-count 1 \
  --node-vm-size Standard_B2s \
  --tier free \
  --enable-managed-identity \
  --generate-ssh-keys
```

> ⏱️ A criação leva alguns minutos. É normal.

---

### 5. Configurar acesso ao cluster

```bash
az aks get-credentials --resource-group $RG --name $CLUSTER --overwrite-existing

# Validar conexão
kubectl get nodes
```

Saída esperada (1 node `Ready`):

```text
NAME                                STATUS   ROLES   AGE   VERSION
aks-nodepool1-xxxxxxxx-vmss000000   Ready    <none>  2m    v1.xx.x
```

Veja a capacidade real do node (vamos comparar com a reserva depois):

```bash
kubectl describe node | grep -A6 "Allocatable:"
```

> Anote o valor de `cpu` em `Allocatable` (algo perto de `1900m` num `Standard_B2s`). É o total de CPU que os pods podem reservar nesse node.

---

### 6. Confirmar o Metrics Server (base do FinOps)

O **Metrics Server** já vem habilitado no AKS. Ele é a fonte do `kubectl top`, que vamos usar para comparar **uso real x recurso reservado**.

```bash
# Confirmar que o metrics-server está rodando
kubectl get pods -n kube-system -l k8s-app=metrics-server

# Ver consumo dos nodes
kubectl top nodes
```

> Se `kubectl top` retornar `<unknown>` logo após criar o cluster, aguarde ~1 minuto para o metrics-server coletar dados e tente novamente.

---

### 7. Criar o manifesto da aplicação — SUPERDIMENSIONADO ❌

Aqui está o ponto central do lab. Vamos definir `requests` e `limits` **propositalmente altos**, como se tivéssemos "chutado para cima por segurança". Cada réplica vai **reservar muito** recurso.

```bash
cat > api-cep.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-cep
  labels:
    app: api-cep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-cep
  template:
    metadata:
      labels:
        app: api-cep
    spec:
      containers:
      - name: api-cep
        image: iesodias/java-api:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "500m"      # reserva 0.5 de CPU POR RÉPLICA -> exagerado
            memory: "700Mi"  # reserva alta para o uso real da app
          limits:
            cpu: "700m"
            memory: "900Mi"
EOF
```

> 🔎 **O que acabamos de fazer?**
> Com `replicas: 2` e `requests.cpu: 500m`, reservamos **1000m de CPU**. E com `requests.memory: 700Mi`, reservamos **1400Mi de memória**. Isso ainda é exagerado para a aplicação, mas evita que o lab quebre cedo demais por causa do overhead do próprio AKS no node.

---

### 8. Fazer o deploy

```bash
kubectl apply -f api-cep.yaml
```

---

### 9. Validar se a aplicação subiu

```bash
# Acompanhe até os pods ficarem Running
kubectl get pods -w
```

Suba um serviço para testar (e usar depois):

```bash
kubectl expose deployment api-cep \
  --name=api-cep-service \
  --type=LoadBalancer \
  --port=80 \
  --target-port=8080
```

Pegue o IP público (pode levar 1-2 min):

```bash
kubectl get service api-cep-service
```

Teste a API (troque `<EXTERNAL-IP>` pelo IP retornado):

```bash
curl http://<EXTERNAL-IP>/api/cep/30350210
curl http://<EXTERNAL-IP>/api/status
```

---

### 10. Troubleshooting — provar o desperdício 🔍

Agora vamos investigar como um SRE/FinOps faria. O objetivo é mostrar o **abismo entre o que foi reservado e o que é realmente usado**.

**10.1 — Uso REAL da aplicação:**

```bash
kubectl top pod -l app=api-cep
```

> Se aparecer `metrics not available yet`, aguarde 30-60 segundos e rode o comando novamente.

Repare: a API consome muito pouco (geralmente alguns `m` de CPU e dezenas de `Mi` de memória):

```text
NAME                       CPU(cores)   MEMORY(bytes)
api-cep-xxxxxxxxxx-aaaaa    6m           90Mi
api-cep-xxxxxxxxxx-bbbbb    5m           88Mi
```

**10.2 — O que foi RESERVADO (requests):**

```bash
kubectl describe deployment api-cep | grep -A8 "Requests:"
```

```text
    Requests:
      cpu:        500m
      memory:     700Mi
```

➡️ **Compare:** reservamos `500m`, mas usamos `~6m`. Estamos desperdiçando quase toda a CPU reservada em cada pod.

**10.3 — Quanto do node está "comprometido" pela reserva:**

```bash
kubectl describe node | grep -A6 "Allocated resources:"
```

```text
  Resource           Requests      Limits
  --------           --------      ------
  cpu                1050m (55%)   1450m (76%)
  memory             1548Mi (41%)  ...
```

➡️ O node aparece com boa parte da capacidade já **comprometida por reserva**... mas o `kubectl top nodes` mostra o uso real lá embaixo:

```bash
kubectl top nodes
```

```text
NAME                                CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
aks-nodepool1-xxxxxxxx-vmss000000   85m          4%     950Mi           24%
```

**Node com reserva alta, mas só 4% usado.** Esse é o desperdício de FinOps na veia.

**10.4 — A prova final: o desperdício impede novos pods.**

Vamos tentar escalar para **4 réplicas**. Como cada uma quer reservar `500m` e `700Mi`, pelo menos uma das novas réplicas deve ficar `Pending` por falta de recurso reservável (na prática, normalmente `Insufficient cpu`) — mesmo com o node praticamente ocioso.

```bash
kubectl scale deployment api-cep --replicas=4
kubectl get pods -l app=api-cep
```

```text
NAME                       READY   STATUS    RESTARTS   AGE
api-cep-xxxxxxxxxx-aaaaa    1/1     Running   0          5m
api-cep-xxxxxxxxxx-bbbbb    1/1     Running   0          5m
api-cep-xxxxxxxxxx-ccccc    1/1     Running   0          20s
api-cep-xxxxxxxxxx-ddddd    0/1     Pending   0          20s
```

Veja o motivo:

```bash
# 1) Copie o nome do pod que ficou Pending no comando anterior
# 2) Rode o describe nesse pod
kubectl describe pod <NOME_DO_POD_PENDING>
```

```text
  Warning  FailedScheduling  ... Insufficient cpu.
```

➡️ Em produção, esse `Insufficient cpu` é o gatilho para **adicionar mais nodes** (Cluster Autoscaler) — ou seja, **gastar mais dinheiro** para resolver um problema que era só reserva inflada.

---

### 11. Diagnóstico (o que os sintomas significam)

| Sintoma observado | Causa raiz | Conclusão FinOps |
|---|---|---|
| `top` mostra uso muito menor que `requests` | `requests` inflado | Pagando por recurso parado |
| Node com alta reserva e baixo uso real | Reserva >> uso real | Node ocioso e caro |
| Nova réplica em `Pending` (`Insufficient cpu`) | Reserva ocupa o node | Pressão para criar nodes a mais |

➡️ **Veredito:** o pod está **superdimensionado**. Não há instabilidade — há **desperdício**. Vamos fazer o **right-sizing para baixo**.

---

### 12. Right-Sizing — reduzir os recursos ✅

Agora ajustamos os `requests`/`limits` para perto do **consumo real** (visto no `kubectl top`) + uma folga de segurança.

Vimos uso de ~`6m` CPU e ~`90Mi` de memória. Agora vamos **editar o mesmo arquivo `api-cep.yaml`** com valores menores:

```bash
cat > api-cep.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-cep
  labels:
    app: api-cep
spec:
  replicas: 4
  selector:
    matchLabels:
      app: api-cep
  template:
    metadata:
      labels:
        app: api-cep
    spec:
      containers:
      - name: api-cep
        image: iesodias/java-api:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "50m"       # ~8x o uso observado, com folga -> realista
            memory: "128Mi"  # folga sobre os ~90Mi usados
          limits:
            cpu: "250m"      # teto que absorve picos
            memory: "256Mi"
EOF
```

> 🎯 **A lógica do right-sizing:** não chutamos "para cima". Olhamos o consumo real no `kubectl top`, adicionamos uma folga e ajustamos. Reservar `50m` em vez de `500m` libera **90% da CPU** por pod — sem perder estabilidade.

Aplicar a correção:

```bash
kubectl apply -f api-cep.yaml
kubectl rollout status deployment/api-cep
```

---

### 13. Revalidar — confirmar a economia

```bash
# 1) As 4 réplicas agora sobem (nada de Pending) -> antes não cabia!
kubectl get pods -l app=api-cep

# 2) O node agora está com a reserva proporcional ao uso real
kubectl describe node | grep -A6 "Allocated resources:"

# 3) A aplicação continua respondendo normalmente
curl http://<EXTERNAL-IP>/api/status
```

Compare o antes e o depois da reserva de CPU no node:

| Cenário | `requests.cpu` por pod | Réplicas | CPU reservada total | Cabe no node (~1900m)? |
|---|---|---|---|---|
| Superdimensionado | `500m` | 4 | `2000m` | Uma réplica fica `Pending` |
| Right-sized | `50m` | 4 | `200m` (~11%) | Sobra recurso para muito mais |

➡️ Mesma aplicação, mesma estabilidade, **uma fração do recurso reservado**. Em escala, isso significa **menos nodes** e **menos fatura**. 🎉

---

### 14. Limpeza (IMPORTANTE para não gerar custo) 🧹

Ao terminar o lab, **apague tudo**. Deletar o Resource Group remove o cluster, o LoadBalancer e o disco de uma vez.

```bash
az group delete --name $RG --yes --no-wait
```

> O `--no-wait` libera seu terminal; a exclusão continua no Azure em segundo plano.

---

### Conclusão

Você praticou o ciclo completo de FinOps em Kubernetes:

- **Mediu** o consumo real com `kubectl top` e o Metrics Server.
- **Diagnosticou** o **superdimensionamento**: muita reserva, pouco uso, node ocioso e pods em `Pending`.
- **Corrigiu** com **right-sizing para baixo**, liberando recurso reservado sem perder estabilidade.

> 🧠 **Lição FinOps:** reservar "por segurança" muito acima do uso **não traz estabilidade — traz fatura**. Barato e estável é reservar **exatamente o que a aplicação precisa**, com uma folga sensata.
