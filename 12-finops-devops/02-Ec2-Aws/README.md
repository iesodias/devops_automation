## Lab 2: FinOps no EC2 (AWS) — Liga/Desliga Programado para Homologação

### Objetivo

Neste lab você vai aprender, na prática, uma das práticas mais simples e eficientes de **FinOps em EC2**: **não deixar ambiente de homologação ligado fora do horário de uso**.

A ideia é simples:

- Criar uma instância EC2 para um ambiente de homologação.
- Validar que ela está rodando.
- Configurar uma rotina automática para:
  - **desligar na sexta-feira à noite**
  - **ligar na segunda-feira de manhã**
- Reduzir custo sem mudar aplicação, sem deploy e sem refatoração.

Esse é um ótimo exemplo de FinOps porque, em muitos times, o ambiente de homologação fica ligado **24x7 sem necessidade**.

> 💡 **Sobre custos:** instância parada não cobra computação, mas continua cobrando recursos como EBS. Mesmo assim, o desligamento programado costuma gerar boa economia.

---

### Pré-requisitos

- `aws cli` instalado e configurado (`aws configure`)
- Permissões para EC2, IAM, EventBridge Scheduler e SSM
- Região AWS ativa (exemplo: `us-east-1`)

```bash
aws --version
aws sts get-caller-identity
```

---

### 1. Criar estrutura local

```bash
mkdir workspace-finops-ec2
cd workspace-finops-ec2
```

---

### 2. Definir variáveis

```bash
export REGION="us-east-1"
export INSTANCE_TYPE="t3.micro"
export INSTANCE_NAME="finops-hml-ec2"
export SCHEDULE_ROLE_NAME="finops-hml-scheduler-role"
export STOP_SCHEDULE_NAME="finops-stop-friday"
export START_SCHEDULE_NAME="finops-start-monday"
```

> Aqui vamos usar `t3.micro` porque o foco do lab não é right-sizing, e sim **economia com agenda de funcionamento**.

---

### 3. Obter AMI mais recente (Amazon Linux 2023)

```bash
export AMI_ID=$(aws ssm get-parameters \
  --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
  --query 'Parameters[0].Value' \
  --output text \
  --region $REGION)

 echo $AMI_ID
```

---

### 4. Usar o Security Group padrão da VPC default

```bash
export DEFAULT_SG_ID=$(aws ec2 describe-security-groups \
  --filters Name=group-name,Values=default \
  --query 'SecurityGroups[0].GroupId' \
  --output text \
  --region $REGION)

 echo $DEFAULT_SG_ID
```

---

### 5. Criar a instância EC2 de homologação

```bash
export INSTANCE_ID=$(aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type $INSTANCE_TYPE \
  --security-group-ids $DEFAULT_SG_ID \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=finops-hml-ec2},{Key=Environment,Value=homologacao}]' \
  --query 'Instances[0].InstanceId' \
  --output text \
  --region $REGION)

 echo $INSTANCE_ID
```

---

### 6. Aguardar running e validar

```bash
aws ec2 wait instance-running --instance-ids $INSTANCE_ID --region $REGION

aws ec2 describe-instances \
  --instance-ids $INSTANCE_ID \
  --query 'Reservations[0].Instances[0].[InstanceType,State.Name]' \
  --output table \
  --region $REGION
```

Esperado: instância em `running`.

---

### 7. Criar a role usada pelo EventBridge Scheduler

O Scheduler precisa de uma role para executar `StartInstances` e `StopInstances` em seu nome.

```bash
cat > scheduler-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "scheduler.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name $SCHEDULE_ROLE_NAME \
  --assume-role-policy-document file://scheduler-trust-policy.json
```

Agora a policy com permissão de ligar e desligar instância:

```bash
cat > scheduler-inline-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam put-role-policy \
  --role-name $SCHEDULE_ROLE_NAME \
  --policy-name finops-hml-ec2-policy \
  --policy-document file://scheduler-inline-policy.json
```

Guarde o ARN da role:

```bash
export SCHEDULE_ROLE_ARN=$(aws iam get-role \
  --role-name $SCHEDULE_ROLE_NAME \
  --query 'Role.Arn' \
  --output text)

 echo $SCHEDULE_ROLE_ARN
```

---

### 8. Criar agenda para desligar na sexta à noite

Exemplo didático: desligar toda **sexta-feira às 20:00**, no fuso de São Paulo.

```bash
aws scheduler create-schedule \
  --name $STOP_SCHEDULE_NAME \
  --schedule-expression "cron(0 20 ? * FRI *)" \
  --schedule-expression-timezone "America/Sao_Paulo" \
  --flexible-time-window '{"Mode":"OFF"}' \
  --target "{\"RoleArn\":\"$SCHEDULE_ROLE_ARN\",\"Arn\":\"arn:aws:scheduler:::aws-sdk:ec2:stopInstances\",\"Input\":\"{\\\"InstanceIds\\\":[\\\"$INSTANCE_ID\\\"]}\"}" \
  --region $REGION
```

**Opcional (via Console AWS):**

Se preferir criar pela interface, em **Amazon EventBridge > Scheduler > Create schedule**, use estes parâmetros:

- `Schedule name`: `finops-stop-friday`
- `Schedule group`: `default`
- `Schedule pattern`: `Recurring schedule`
- `Schedule type`: `Cron-based schedule`
- `Cron expression`: `cron(0 20 ? * FRI *)`
- `Time zone`: `America/Sao_Paulo`
- `Flexible time window`: `Off`
- `Target type`: `Templated targets` (ou `Universal target`, se essa for a opção exibida)
- `Target service/API`: `EC2 - StopInstances`
- `Input payload`: `{"InstanceIds":["<INSTANCE_ID>"]}`
- `Execution role`: usar a role criada no passo 7 (`$SCHEDULE_ROLE_ARN`)
- `Retry policy` e `Dead-letter queue`: manter padrão para este lab

---

### 9. Criar agenda para ligar na segunda de manhã

Exemplo didático: ligar toda **segunda-feira às 08:00**, no mesmo fuso.

```bash
aws scheduler create-schedule \
  --name $START_SCHEDULE_NAME \
  --schedule-expression "cron(0 8 ? * MON *)" \
  --schedule-expression-timezone "America/Sao_Paulo" \
  --flexible-time-window '{"Mode":"OFF"}' \
  --target "{\"RoleArn\":\"$SCHEDULE_ROLE_ARN\",\"Arn\":\"arn:aws:scheduler:::aws-sdk:ec2:startInstances\",\"Input\":\"{\\\"InstanceIds\\\":[\\\"$INSTANCE_ID\\\"]}\"}" \
  --region $REGION
```

**Opcional (via Console AWS):**

Se preferir criar pela interface, em **Amazon EventBridge > Scheduler > Create schedule**, use estes parâmetros:

- `Schedule name`: `finops-start-monday`
- `Schedule group`: `default`
- `Schedule pattern`: `Recurring schedule`
- `Schedule type`: `Cron-based schedule`
- `Cron expression`: `cron(0 8 ? * MON *)`
- `Time zone`: `America/Sao_Paulo`
- `Flexible time window`: `Off`
- `Target type`: `Templated targets` (ou `Universal target`, se essa for a opção exibida)
- `Target service/API`: `EC2 - StartInstances`
- `Input payload`: `{"InstanceIds":["<INSTANCE_ID>"]}`
- `Execution role`: usar a role criada no passo 7 (`$SCHEDULE_ROLE_ARN`)
- `Retry policy` e `Dead-letter queue`: manter padrão para este lab

---

### 10. Validar as agendas criadas

```bash
aws scheduler get-schedule --name $STOP_SCHEDULE_NAME --region $REGION
aws scheduler get-schedule --name $START_SCHEDULE_NAME --region $REGION
```

O objetivo aqui não é esperar até segunda ou sexta. O objetivo do lab é mostrar que:

- a instância existe
- a automação foi criada
- a economia acontecerá sem ação manual do time

---

### 11. Diagnóstico FinOps

Esse cenário representa bem um ambiente de homologação:

- usado em horário comercial
- pouco ou nada usado no fim de semana
- frequentemente esquecido ligado

Se você desliga a instância do fim da sexta até a manhã de segunda, você elimina muitas horas de computação desnecessária ao longo do mês.

**Exemplo de raciocínio FinOps:**

- sem automação: ambiente ligado **24x7**
- com automação: ambiente ligado só quando o time realmente usa
- resultado: **economia recorrente**, simples e previsível

---

### 12. Limpeza (obrigatório)

Se quiser encerrar tudo ao final do lab:

```bash
aws scheduler delete-schedule --name $STOP_SCHEDULE_NAME --region $REGION
aws scheduler delete-schedule --name $START_SCHEDULE_NAME --region $REGION

aws ec2 terminate-instances --instance-ids $INSTANCE_ID --region $REGION
aws ec2 wait instance-terminated --instance-ids $INSTANCE_ID --region $REGION

aws iam delete-role-policy \
  --role-name $SCHEDULE_ROLE_NAME \
  --policy-name finops-hml-ec2-policy

aws iam delete-role --role-name $SCHEDULE_ROLE_NAME

rm -f scheduler-trust-policy.json scheduler-inline-policy.json
```

---

### Conclusão

Você executou uma prática FinOps muito comum em ambientes de homologação:

- criou uma EC2 para uso do time
- configurou uma automação de liga/desliga
- reduziu desperdício sem mexer em aplicação
- transformou economia em rotina operacional

> 🧠 **Lição FinOps:** nem toda economia vem de reduzir tamanho. Muitas vezes, a maior economia vem de **não deixar recurso ligado quando ninguém está usando**.
