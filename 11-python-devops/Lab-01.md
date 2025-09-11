# Lab 0 — Instalar Python no Ubuntu, criar virtual env e fazer um Hello World

## 🎯 Objetivo
Preparar o ambiente para rodar Python, criar um ambiente virtual e executar o primeiro script "Hello World".

---

## 🧩 Passo 1 — Conferir a versão do Python
```bash
lsb_release -sd             # Mostra a versão do Ubuntu
python3 --version           # Verifica se o Python 3 já está instalado
which python3               # Mostra o caminho do Python, se existir
```
✅ **Saída esperada:**
```
Ubuntu 22.04 LTS
Python 3.10.12
```
Se o Python não estiver instalado, siga para o próximo passo.

---

## 🧩 Passo 2 — Instalar Python 3, pip e venv
```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip
# opcional: usar "python" no lugar de "python3"
sudo apt install -y python-is-python3
```
**Verifique as versões:**
```bash
python --version
pip --version
```

---

## 🧩 Passo 3 — Criar pasta do projeto
```bash
mkdir -p ~/labs/python-hello
cd ~/labs/python-hello
pwd
```
📌 **Dica:** mantenha uma pasta dedicada para seus labs.

---

## 🧩 Passo 4 — Criar e ativar o virtual env
```bash
python3 -m venv .venv
source .venv/bin/activate
```
✅ **Saída esperada:**
```
(.venv) usuario@ubuntu:~/labs/python-hello$
```
**Confirme se está no ambiente virtual:**
```bash
which python
python -V
pip -V
```

---

## 🧩 Passo 5 — Criar o arquivo "Hello World"
Crie o arquivo usando o comando `touch`:
```bash
touch main.py
```
Agora abra o arquivo no editor de texto que preferir (`nano`, `vim` ou outro). Exemplo usando `nano`:
```bash
nano main.py
```
Digite o conteúdo abaixo:
```python
print("Hello, DevOps!")
```
Salve e feche o arquivo. No `nano`, use:
- `Ctrl + O` → salva o arquivo
- `Ctrl + X` → sai do editor

Execute o script:
```bash
python main.py
```
✅ **Saída esperada:**
```
Hello, DevOps!
```

---

## 🧩 Passo 6 — Testar o REPL (interpretador)
```bash
python
```
No interpretador:
```python
print("Hello, DevOps do REPL!")
exit()
```

---

## 🧩 Passo 7 — Checkpoint do ambiente
```bash
python - << 'PY'
import sys, platform
print("Python:", sys.version.split()[0])
print("SO:", platform.system(), platform.release())
PY
```
✅ **Saída esperada:**
```
Python: 3.10.12
SO: Linux 5.15.0
```

---

## 🧩 Passo 8 — Desativar o virtual env
```bash
deactivate
```
📌 **Dica:** o comando `deactivate` volta ao Python global.

---

## 🛠 Problemas comuns e soluções
- **Erro:** `source .venv/bin/activate: No such file or directory`
  - 🔹 Solução: recrie o venv com `python3 -m venv .venv`
- **Erro:** `python: command not found`
  - 🔹 Solução: instale o alias com `sudo apt install python-is-python3`
- **Erro de permissão ao instalar pacotes**
  - 🔹 Solução: ative o venv e instale dentro dele

---

## 🚀 O que você aprendeu
- Instalar e configurar Python 3
- Criar e ativar um ambiente virtual
- Criar um arquivo manualmente e rodar seu primeiro script
- Usar o interpretador para testar comandos

---

Pronto! Agora temos o ambiente Python preparado para os próximos labs 😎
