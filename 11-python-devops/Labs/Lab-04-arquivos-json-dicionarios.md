# Lab-04 — Arquivos, JSON e Dicionários

## Objetivo

Aprender a trabalhar com dicionários, leitura/escrita de arquivos e JSON no Python.

---

## Passo 1 — Preparar o ambiente

```bash
mkdir -p ~/labs/python-arquivos
cd ~/labs/python-arquivos
python3 -m venv .venv
source .venv/bin/activate
```

---

## Passo 2 — Entendendo Dicionários

Abra o interpretador:
```bash
python
```

Digite os comandos:
```python
servidor = {
    "nome": "web-server-01",
    "ip": "192.168.1.100",
    "porta": 8080,
    "ativo": True
}

# Acessando valores
print(servidor["nome"])
print(servidor.get("regiao", "nao definida"))

# Adicionando nova chave
servidor["ambiente"] = "producao"

# Listando chaves e valores
print(servidor.keys())
print(servidor.values())

exit()
```

---

## Passo 3 — Lista de Dicionários

Crie o arquivo `servidores.py`:
```bash
touch servidores.py
nano servidores.py
```

Conteudo:
```python
servidores = [
    {"nome": "web-01", "ip": "10.0.0.1", "cpu": 45},
    {"nome": "web-02", "ip": "10.0.0.2", "cpu": 78},
    {"nome": "db-01", "ip": "10.0.0.3", "cpu": 92},
]

for srv in servidores:
    if srv["cpu"] >= 90:
        status = "CRITICAL"
    elif srv["cpu"] >= 70:
        status = "WARNING"
    else:
        status = "OK"
    
    print(f"{srv['nome']} | CPU: {srv['cpu']}% | {status}")
```

Execute:
```bash
python servidores.py
```

---

## Passo 4 — Leitura e Escrita de Arquivos

Crie o arquivo `arquivos.py`:
```bash
touch arquivos.py
nano arquivos.py
```

Conteudo:
```python
# Escrita (modo 'w' sobrescreve)
with open("log.txt", "w") as f:
    f.write("2024-01-15 10:00:00 INFO App iniciada\n")
    f.write("2024-01-15 10:01:00 ERROR Falha no DB\n")
    f.write("2024-01-15 10:02:00 INFO Reconectado\n")

print("Arquivo log.txt criado!")

# Leitura completa
with open("log.txt", "r") as f:
    print(f.read())

# Leitura linha por linha
with open("log.txt", "r") as f:
    for linha in f:
        if "ERROR" in linha:
            print("ERRO:", linha.strip())

# Append (modo 'a' adiciona ao final)
with open("log.txt", "a") as f:
    f.write("2024-01-15 10:03:00 INFO Nova linha\n")
```

Execute:
```bash
python arquivos.py
```

---

## Passo 5 — Trabalhando com JSON

Crie o arquivo `json_demo.py`:
```bash
touch json_demo.py
nano json_demo.py
```

Conteudo:
```python
import json

config = {
    "app_name": "api-gateway",
    "port": 8080,
    "debug": False,
    "hosts": ["localhost", "api.exemplo.com"]
}

# Salvar em arquivo
with open("config.json", "w") as f:
    json.dump(config, f, indent=2)

print("config.json salvo!")

# Ler do arquivo
with open("config.json", "r") as f:
    dados = json.load(f)

print(f"App: {dados['app_name']}")
print(f"Porta: {dados['port']}")

# Modificar e salvar
dados["version"] = "2.0.0"
with open("config.json", "w") as f:
    json.dump(dados, f, indent=2)
```

Execute:
```bash
python json_demo.py
cat config.json
```

---

## Passo 6 — Mini-Projeto: Analisador de Logs

Crie o arquivo `analisador.py`:
```bash
touch analisador.py
nano analisador.py
```

Conteudo:
```python
import json

# Criar log de exemplo
logs = """INFO App iniciada
ERROR Falha no banco
WARNING CPU alta
ERROR Timeout API
INFO Recuperado
"""

with open("app.log", "w") as f:
    f.write(logs)

# Analisar
stats = {"info": 0, "warning": 0, "error": 0}

with open("app.log", "r") as f:
    for linha in f:
        if "INFO" in linha:
            stats["info"] += 1
        elif "WARNING" in linha:
            stats["warning"] += 1
        elif "ERROR" in linha:
            stats["error"] += 1

print("=== Relatorio ===")
print(f"INFO: {stats['info']}")
print(f"WARNING: {stats['warning']}")
print(f"ERROR: {stats['error']}")

# Salvar relatorio
with open("relatorio.json", "w") as f:
    json.dump(stats, f, indent=2)

print("Relatorio salvo em relatorio.json")
```

Execute:
```bash
python analisador.py
cat relatorio.json
```

---

## O que voce aprendeu

- Dicionarios: estrutura chave-valor
- Lista de dicionarios para representar colecoes
- Leitura e escrita de arquivos com `open()` e `with`
- Manipulacao de JSON com `json.dump()` e `json.load()`
- Projeto pratico: analisador de logs
