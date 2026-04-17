# A Mágica do Painel Administrativo do Django

Se você entrou no painel agora, deve ter pensado: *"Ok, tem uma tela verde bonita com Usuários e Grupos... E daí? Pra que isso serve?"*

A verdadeira **mágica** do Django acontece quando você cria as **suas próprias coisas** (tabelas de banco de dados, que no Django chamamos de `Models`). Hoje, vamos ensinar o Django a gerenciar uma loja virtual. Vamos criar uma tabela de "Produtos" e você vai ver como o painel admin brilha!

---

## Passo 1: Criando a nossa primeira Tabela (Model)

No Django, você não precisa saber a linguagem complexa de banco de dados (SQL). Você apenas escreve código em Python e o Django traduz tudo sozinho.

1. No terminal do servidor (lembre-se de estar com a bolha `(venv)` ativada!), abra o arquivo responsável pelas tabelas do seu App:

```bash
nano core/models.py
```

2. Apague o que estiver lá e cole este código:

```python
from django.db import models

class Produto(models.Model):
    nome = models.CharField(max_length=100) # Um texto curto
    preco = models.DecimalField(max_digits=8, decimal_places=2) # Dinheiro, ex: 150.50
    em_estoque = models.BooleanField(default=True) # Verdadeiro ou Falso (Caixinha de marcar)

    # Isso serve para o item aparecer com o nome dele no painel de controle (ex: "Camiseta" em vez de "Objeto 1")
    def __str__(self):
        return self.nome
```

3. Salve e saia do nano (`Ctrl + O`, `Enter`, `Ctrl + X`).

---

## Passo 2: Avisando o Banco de Dados (Migrações)

Sempre que você cria uma classe nova no `models.py`, o banco de dados não fica sabendo automaticamente. Nós precisamos mandar o mordomo `manage.py` fabricar a planta da tabela e construir de fato o cimento no banco de dados.

1. **Faça a planta (makemigrations):**
```bash
python3 manage.py makemigrations
```
*(O terminal vai avisar com uma mensagem verde que criou o modelo Produto)*

2. **Cimente a tabela no banco (migrate):**
```bash
python3 manage.py migrate
```
*(Ele vai aplicar essa planta no banco de dados de verdade)*

---

## Passo 3: O Grande Truque de Mágica (Registrando no Admin)

Agora que a tabela `Produto` existe, nós só precisamos dar o "convite VIP" para ela aparecer no Painel de Administração. É literalmente **uma linha de código**!

1. Abra o arquivo do painel administrativo do seu aplicativo:
```bash
nano core/admin.py
```

2. Altere para ficar exatamente assim:
```python
from django.contrib import admin
from .models import Produto   # 1. Importando a tabela que acabamos de criar

admin.site.register(Produto)  # 2. O truque de mágica: "Ei Admin, cuide dessa tabela pra mim!"
```
3. Salve e saia do nano (`Ctrl + O`, `Enter`, `Ctrl + X`).

---

## Passo 4: Acessando o Painel e Vendo a Mágica

Se o seu servidor não estiver rodando, ligue-o novamente:
```bash
python3 manage.py runserver 0.0.0.0:8000
```

1. Acesse novamente o painel no seu navegador: `http://SEU_IP:8000/admin`
2. Faça login com o usuário que você criou com o `cratersuperuser`.
3. Olhe para a tela! Você vai ver uma nova seção verde chamada **CORE** e, lá dentro, **Produtos**.

### O que você ganhou escrevendo apenas aquelas 10 linhas de código no Passo 1 e o Passo 3?

Clique no botão **"+ Add"** (Adicionar) ao lado de *Produtos*.

**O Django acabou de construir do zero para você:**
- Uma tela HTML estilizada e segura para cadastrar produtos.
- Um formulário perfeito (o campo `nome` é um texto, o campo `preco` só aceita números e ponto, o `em_estoque` virou uma caixinha de marcar).
- Validação automática: Tente salvar o preço escrevendo "ABC" dentro — ele não vai deixar e vai te dar um erro em vermelho.
- Um botão de deletar seguro (que pergunta "Você tem certeza?" antes de apagar).
- Uma tela de listar itens e buscar por eles.

Na maioria das outras linguagens de programação, você demoraria um dia inteiro (ou mais) desenhando o HTML dessa tela, fazendo as rotas ("Views") do painel, criando a segurança da rota, validando os dados, enviando dados via requisição, e finalmente salvando no SQL. **No Django, você fez tudo isso em menos de 5 minutos.**

Essa é a serventia do painel Admin! Ele é o painel de controle central de toda a base de dados do seu projeto, gerado 100% automaticamente.

---

## Passo 5: O Ciclo Completo (Exibindo Produtos na Página Inicial)

No seu primeiro tutorial, nós criamos uma página inicial estática que só dizia *"Olá Mundo"*. Agora, nós vamos fazer com que ela vá correndo lá no banco de dados, pegue todos os produtos que você acabou de cadastrar no Admin e os exiba na tela! Isso ajuda a fechar o ciclo do Django (Model, Template, View).

Lá no arquivo **`core/views.py`**, altere o código para ficar assim:

```python
from django.shortcuts import render
from django.http import HttpResponse

# 1. IMPORTANTE: Fale para a sua View que a tabela Produto existe
from .models import Produto 

def pagina_inicial(request):
    # 2. Pega todos os produtos salvos no banco de dados (A mágica!)
    meus_produtos = Produto.objects.all()
    
    # 3. Montar um texto inicial (HTML bruto)
    html_da_pagina = "<h1>Bem-vindo à nossa Lojinha no Amazon Linux!</h1><h2>Nossos Produtos:</h2>"

    # 4. Fazer um "loop" para adicionar cada produto na tela
    for item in meus_produtos:
        html_da_pagina += f"<p>➡️ {item.nome} - R$ {item.preco}</p>"
    
    # 5. Entregar a resposta final montada para o usuário ver
    return HttpResponse(html_da_pagina)
```

**O que vai acontecer?**
Se você salvar esse arquivo e atualizar a aba da página inicial (`http://SEU_IP:8000/`) no seu navegador, todo texto verde novo que você cadastrar lá no painel Admin aparecerá **automaticamente** na página principal do seu site!

Você acabou de construir um sistema completo conectando banco de dados, controle de estoque (Admin) e vitrine pública (View)! Parabéns!
