# Lab-03 Debug no PyCharm - Def - For - List

---

## Objetivo

* Ensinar do zero como criar e entender funções no Python
* Mostrar como listas e laços `for` funcionam
* Aprender a rodar um código simples no PyCharm
* Usar o **debug** para enxergar variáveis mudando em tempo real

---

## O que é uma função (`def` no Python)

* Função = bloco de código que executa uma tarefa específica
* Palavra-chave `def` = usada para **definir** uma função

**Exemplo simples:**

```python
def cumprimentar(nome):
    return f"Olá, {nome}!"

print(cumprimentar("Ieso Dias"))
```

---

## O que é uma lista

* Lista = uma **caixa que guarda vários valores** ao mesmo tempo
* Escrita entre colchetes `[]`

**Exemplo:**

```python
numeros = [3, 7, 2, 9, 5]
print(numeros)       # mostra a lista completa
print(numeros[0])    # mostra o primeiro número da lista (3)
print(len(numeros))  # mostra quantos números tem (5)
```

A lista guarda vários números e podemos acessar cada posição.

---

## O que é o laço `for`

* `for` = comando para **repetir uma ação** para cada item de uma lista

**Exemplo:**

```python
frutas = ["maçã", "banana", "laranja"]

for fruta in frutas:
    print("Eu gosto de", fruta)
```

Esse código vai imprimir uma frase para cada fruta.

Saída:

```
Eu gosto de maçã
Eu gosto de banana
Eu gosto de laranja
```

---

## Passo 1 — Criar o arquivo no PyCharm

1. Abra o **PyCharm**
2. Clique em **File → New Project**
3. Dê um nome para o projeto (ex: `lab_debug`)
4. Dentro do projeto, clique com o direito na pasta → **New → Python File**
5. Nomeie como `lab_debug.py`
6. Cole este código:

```python
def soma_lista(numeros):
    total = 0
    for n in numeros:
        total += n
    return total


def maior_numero(numeros):
    maior = numeros[0]
    for n in numeros:
        if n > maior:
            maior = n
    return maior


def main():
    lista = [3, 7, 2, 9, 5]
    resultado_soma = soma_lista(lista)
    resultado_maior = maior_numero(lista)

    print("Lista:", lista)
    print("Soma:", resultado_soma)
    print("Maior número:", resultado_maior)


if __name__ == "__main__":
    main()
```

---

## Passo 2 — Executar o programa

1. Clique com o botão direito no arquivo `lab_debug.py`
2. Escolha **Run 'lab_debug'**
3. Veja a saída no console:

```
Lista: [3, 7, 2, 9, 5]
Soma: 26
Maior número: 9
```

Isso mostra que o programa somou todos os números e encontrou o maior.

---

## Passo 3 — O que é um breakpoint

* Breakpoint = ponto de parada no código
* Serve para **congelar** a execução e observar variáveis

Clique na lateral esquerda da linha do código (gutter) para adicionar.

Coloque breakpoints em:

* `total += n`
* `if n > maior:`
* `print("Soma:", resultado_soma)`

---

## Passo 4 — Rodar em modo Debug

1. Clique com o botão direito no arquivo → **Debug 'lab_debug'**
2. O programa vai parar no **primeiro breakpoint**
3. Use a aba **Variables** para ver os valores mudando

Exemplo: dentro de `soma_lista`, o valor de `total` começa em 0 e aumenta a cada volta do `for`.

---

## Passo 5 — Botões principais do Debug

* **Step Over (F8):** vai para a próxima linha sem entrar em funções
* **Step Into (F7):** entra dentro da função chamada
* **Step Out (Shift+F8):** sai da função e volta para a principal
* **Resume (F9):** continua até o próximo breakpoint

Esses botões deixam você controlar o ritmo da execução.

---

## Passo 7 — Conclusão

* Aprendemos que `def` cria funções
* Listas guardam vários valores
* O `for` percorre cada item de uma lista
* Breakpoints deixam o programa parar para observar variáveis
* O debug torna visível o que acontece dentro do código

