# Tutorial Completo (Parte 4): Sistema de Busca e Filtros Inteligentes

Neste tutorial vamos evoluir nossa página inicial adicionando a "Cereja do Bolo" clássica de todo site: Uma barra de busca! Com isso, os clientes não precisarão rolar páginas infinitas para encontrar o que desejam. Ao digitar e pressionar a busca, a URL entregará o comando público para o Cérebro Python no servidor EC2, que filtrará velozmente o Banco de Dados em tempo real.

---

## Passo 1: Atualizando o Motor Lógico (`core/views.py`)

A mágica da busca acontece alterando pouquíssimas linhas. Nosso arquivo `views.py` vai tentar capturar e "pescar" a palavra que o usuário preencheu enviando na barra de endereço usando o método `GET`.

Abra o arquivo (`nano core/views.py`) e altere-o por este código completo:

```python
from django.shortcuts import render
from .models import Produto

def pagina_inicial(request):

    # 1. Trazemos absolutamente todos os produtos primeiro (A vitrine padrão)
    produtos_do_banco = Produto.objects.all()

    # 2. Pescamos silenciosamente se o botão "Buscar" do HTML foi acionado:
    termo_digitado = request.GET.get('busca_do_usuario')

    # 3. Mágica do Banco de Dados: Se houver uma palavra capturada ali em cima...
    if termo_digitado:
        # Pedimos pro Banco AWS: "Filtre SÓ quem contém a Palavra (ignorando maiúsculas e minúsculas)"
        # O icontains de nome__icontains significa: 'Insensitive Case Contains' (Contém ignorando letras maiúsculas/minúsculas)
        produtos_do_banco = produtos_do_banco.filter(nome__icontains=termo_digitado)

    # 4. Empacota pra viagem! (Juntamos isso com aquele pacotão final)
    pacote_de_dados = {
        'nome_astronauta': "Viajante do Espaço",
        'acessos_totais': 42,
        'linguagem_secreta': "Python e Mágica",

        # Enviamos a lista que agora pode estar cheia (Padrão) OU espremida pela Filtragem acima!
        'lista_de_produtos': produtos_do_banco,

        # Devolvemos a palavra para a tela! Assim a caixa de texto do cliente não limpa irritando-o.
        'palavra_pesquisada': termo_digitado,
    }

    return render(request, 'core/index.html', pacote_de_dados)
```

_(Salve e feche o arquivo com `Ctrl+O`, `Enter`, `Ctrl+X`)._

---

## Passo 2: O Visual Front-end (`core/templates/core/index.html`)

Agora precisaremos acrescentar a nossa nova Barra no leiaute CSS! Ela será um botão formador do tráfego através de `<form>`. E, pensando já no design final profissional, também vamos avisar educadamente e com uma carinha triste caso o cliente faça uma busca tão insana em que o sistema retorne 0 resultados do estoque!

Abra `nano core/templates/core/index.html` e cole o código gigante definitivo:

```html
<!DOCTYPE html>
<html lang="pt-br">
  <head>
    <meta charset="UTF-8" />
    <title>Minha Vitrine Amazon EC2 com Busca Inteligente</title>
    <style>
      body {
        font-family: "Segoe UI", Arial, sans-serif;
        background-color: #2c3e50;
        text-align: center;
        padding: 50px;
      }
      .caixa {
        background: #ecf0f1;
        padding: 40px;
        border-radius: 12px;
        box-shadow: 0px 4px 15px rgba(0, 0, 0, 0.3);
        display: inline-block;
        max-width: 800px;
        text-align: left;
        width: 100%;
      }
      h1 {
        color: #2980b9;
        text-align: center;
      }
      p {
        color: #34495e;
        font-size: 18px;
        text-align: center;
      }
      .vitrine {
        margin-top: 30px;
        border-top: 3px solid #bdc3c7;
        padding-top: 20px;
      }
      .produto-card {
        background: white;
        border-left: 5px solid #27ae60;
        padding: 15px;
        margin-bottom: 15px;
        border-radius: 4px;
        box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
      }
      .preco {
        font-size: 22px;
        color: #27ae60;
        font-weight: bold;
      }
      .esgotado {
        border-left-color: #e74c3c;
      }
      .esgotado .preco {
        color: #e74c3c;
      }

      /* NOVO ESTILO BONITO DA BARRA DE BUSCA */
      .area-busca {
        background: #34495e;
        padding: 20px;
        border-radius: 8px;
        margin: 20px 0;
        display: flex;
        gap: 10px;
        justify-content: center;
      }
      .input-busca {
        flex: 1;
        padding: 12px;
        font-size: 16px;
        border: none;
        border-radius: 4px;
        outline: none;
        width: 100%;
      }
      .btn-busca {
        background: #2980b9;
        color: white;
        border: none;
        padding: 12px 25px;
        font-size: 16px;
        border-radius: 4px;
        cursor: pointer;
        font-weight: bold;
        transition: 0.3s;
      }
      .btn-busca:hover {
        background: #1f618d;
      }
      .mensagem-erro {
        text-align: center;
        color: #e74c3c;
        font-size: 20px;
        padding: 30px;
      }
    </style>
  </head>
  <body>
    <div class="caixa">
      <h1>Lojinha do Amazon EC2</h1>

      <!-- ============================================== -->
      <!-- O SEGREDO DO GET: NOSSA BARRA DE BUSCA NA TELA -->
      <!-- ============================================== -->
      <form method="GET" class="area-busca">
        <!-- Aquele campo mágico HTML que engole o nome para a URL -->
        <!-- O default_if_none garante que ele não imprima o termo de programaçao na tela na primeira visita limpa do usuário -->
        <input
          type="text"
          name="busca_do_usuario"
          class="input-busca"
          placeholder="Busque nossos produtos pelo nome..."
          value="{{ palavra_pesquisada|default_if_none:'' }}"
        />
        <button type="submit" class="btn-busca">🔍 Buscar</button>
      </form>

      <div class="vitrine">
        <h2>Nosso Estoque Atual:</h2>

        <!-- INÍCIO DA MÁGICA: VERIFICA SE DEU RESULTADO OU SE TA ZERADO (Len == 0) -->
        {% if lista_de_produtos %} {% for produto in lista_de_produtos %} {% if
        produto.em_estoque %}
        <div class="produto-card">
          <h3 style="margin: 0; color: #2c3e50;">{{ produto.nome }}</h3>
          <div class="preco">R$ {{ produto.preco }}</div>
          <div style="color: #27ae60; font-weight: bold; margin-top: 5px;">
            ✅ Em Estoque
          </div>
        </div>
        {% else %}
        <div class="produto-card esgotado">
          <h3 style="margin: 0; color: #2c3e50;">{{ produto.nome }}</h3>
          <div class="preco">R$ {{ produto.preco }}</div>
          <div style="color: #e74c3c; font-weight: bold; margin-top: 5px;">
            Esgotado
          </div>
        </div>
        {% endif %} {% endfor %} {% else %}

        <!-- E SE O CLIENTE BUSCOU E NÃO ACHOU NADA QUE BATE IGUAL NA LÓGICA DO icontains? O DJANGO CAI AQUI AUTOMATICAMENTE -->
        <div class="mensagem-erro">
          <p>Poxa...</p>
          <p>
            Não encontramos nenhum produto no nosso estoque chamado
            <strong>"{{ palavra_pesquisada }}"</strong>.
          </p>
          <a
            href="/"
            style="display:inline-block; margin-top: 15px; color: #2980b9; text-decoration: none; font-weight:bold;"
            >Limpar Filtro de Busca</a
          >
        </div>

        {% endif %}
        <!-- FIM DO LOOP -->
      </div>
    </div>
  </body>
</html>
```

## Passo 3: Testando na Internet!

Acesse novamente o seu site no Amazon Linux e repare no surgimento da formidável Barra de Busca escura.

**Faça o teste das mensagens sem falhas (Fail Safe):**
Digite algo absurdo que sabemos não existir (ex: "Navio Pirata Marciano") e clique em **Buscar**. Repare que lá em cima na URL o texto alterou magicamente para injetar ao Cérebro via GET: `http://SEU_IP/?busca_do_usuario=Navio+Pirata...`. Como o Django filtrou para zero resultados do Banco, você será saudado com uma bela mensagem de erro triste desenhada puramente por aquele braço de `{% else %}`! Experimente depois buscar um produto normal do seu Painel Admin para o alívio dele voltar lindamente estampado na listagem verde!
