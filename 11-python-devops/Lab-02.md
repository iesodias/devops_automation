# Lab 1 — Fundamentos do Python para DevOps

## 🎯 Objetivo
Aprender os conceitos básicos de Python direto no **interpretador** e em pequenos scripts. Vamos trabalhar com variáveis, tipos, formatação de texto, condicionais, imports, tratamento de erros e um mini-desafio.

---

## 🧩 Passo 1 — Preparar o ambiente
```bash
mkdir -p ~/labs/python-basics
cd ~/labs/python-basics
python3 -m venv .venv
source .venv/bin/activate
```
✅ **Saída esperada:**
```
(.venv) user@ubuntu:~/labs/python-basics$
```

---

## 🧩 Passo 2 — Variáveis, tipos e f-strings
Abra o interpretador:
```bash
python
```
Digite os comandos abaixo **um por vez**:
```python
user = "devops"
age = 39
rate = 0.15
active = True

# Descobrindo os tipos


# Usando f-string para formatar texto
msg = f"User {user} | Age {age} | Active {active}"
msg
```

### Conversão de tipos
```python
int("42")
str(123)
float("3.14")
# Este vai gerar um erro proposital:
int("42.0")
```
Para sair do interpretador:
```python
exit()
```

---

## 🧩 Passo 3 — Condicionais práticas
Crie um arquivo chamado `conditionals.py`:
```bash
touch conditionals.py
nano conditionals.py
```
Digite o código:
```python
cpu = 73  # altere para testar valores diferentes

if cpu >= 90:
    status = "CRITICAL"
elif cpu >= 70:
    status = "HIGH"
else:
    status = "OK"

print(f"CPU {cpu}% | Status {status}")

mem = 82
if cpu >= 70 and mem >= 80:
    print("Warning: High CPU and Memory usage")
else:
    print("Resources within expected range")
```
Salve, saia e execute:
```bash
python conditionals.py
```

---

## 🧩 Passo 4 — Imports úteis para DevOps
Crie um arquivo chamado `imports.py`:
```bash
touch imports.py
nano imports.py
```
Digite o código:
```python
import os, platform
from datetime import datetime



```
Execute:
```bash
python imports.py
```

---

## 🧩 Passo 5 — Tratando erros com try/except
Crie um arquivo chamado `errors.py`:
```bash
touch errors.py
nano errors.py
```
Digite o código:
```python
# Caso válido
try:
    value = int("42")
    result = value * 2
    print(f"OK value={value} result={result}")
except ValueError as e:
    print(f"Error converting: {e}")

# Caso inválido: dispara ValueError
try:
    value = int("42.0")
    result = value * 2
    print(f"OK value={value} result={result}")
except ValueError as e:
    print(f"Error converting: {e}")

# Uso de else e finally
try:
    v = int("100")
except ValueError:
    print("error")
else:
    print(f"Conversion OK v={v}")
finally:
    print("always executes")
```
Execute:
```bash
python errors.py
```

---

## 🧩 Passo 6 — Mini-desafio integrando tudo
Crie um arquivo chamado `challenge.py`:
```bash
touch challenge.py
nano challenge.py
```
Digite o código:
```python
from datetime import datetime
import platform

entrada = input("Enter CPU usage in %: ")  # exemplo: 77

try:
    cpu = int(entrada)

    # Validando valor
    if cpu < 0 or cpu > 100:
        raise ValueError("CPU must be between 0 and 100")

    # Classificando o uso da CPU
    if cpu >= 90:
        status = "CRITICAL"
    elif cpu >= 70:
        status = "HIGH"
    else:
        status = "OK"

    os_name = platform.system()
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"[{now}] OS={os_name} | CPU={cpu}% | Status={status}")
except ValueError as e:
    print(f"Invalid input: {e}")
```
Execute:
```bash
python challenge.py
```
✅ **Teste com diferentes entradas:**
- **Entrada válida:** `85` → Status: **HIGH**
- **Entrada inválida:** `abc` → **Invalid input**

---

## 🚀 O que você aprendeu
- Criar diretórios e organizar arquivos
- Trabalhar com variáveis, tipos e f-strings
- Escrever condicionais e operadores lógicos
- Usar imports úteis para DevOps
- Tratar erros com try/except
- Criar um mini-script para integração de conceitos
