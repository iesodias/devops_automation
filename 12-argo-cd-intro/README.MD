# Lab de Introdução ao ArgoCD no AKS

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
