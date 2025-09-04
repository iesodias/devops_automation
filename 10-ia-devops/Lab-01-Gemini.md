# Passo a passo para instalar o Gemini CLI no Ubuntu

Este guia garante que a versão correta do Node.js, necessária para a ferramenta, seja instalada.

## Passo 1: Atualizar os pacotes do sistema

```bash
sudo apt update
```

## Passo 2: Instalar o curl
*(Necessário para baixar o script do Node.js)*

```bash
sudo apt install curl -y
```

## Passo 3: Adicionar o repositório do Node.js v20.x
*(O repositório padrão do Ubuntu possui uma versão antiga)*

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

## Passo 4: Instalar o Node.js e o NPM
*(Agora o sistema irá instalar a versão recente do repositório adicionado)*

```bash
sudo apt-get install -y nodejs
```

## Passo 5: Instalar o Gemini CLI com o NPM

```bash
sudo npm install -g @google/gemini-cli
```

## Passo 6: Fazer login com sua Conta Google

```bash
gemini auth login
```

Este comando gerará um link. Abra-o no seu navegador para fazer a autenticação e autorizar a CLI.

## Passo 7: Testar a instalação

```bash
gemini "Qual a previsão do tempo para amanhã?"
```