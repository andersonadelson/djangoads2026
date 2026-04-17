# Tutorial Completo (Parte 2): Templates HTML e Variáveis no Django

Você já tem o servidor rodando e respondendo a acessos lá no Amazon Linux. Mas na vida real, os sites não são apenas um texto jogado pelo Python. Eles precisam de cores, caixas organizadas, botões e leiautes complexos construídos em **HTML** e **CSS**.

Nesta segunda etapa do nosso aprendizado, vamos aprender a tirar o código visual de dentro do arquivo Python para não misturar as coisas e transferi-lo para um arquivo **Template HTML**, e o mais fantástico: aprender a enviar "segredos" (variáveis dinâmicas de dentro do banco de dados) direto do cérebro do Python para substituir buracos na tela do usuário!

---

## Passo 1: O Que São os "Templates" no Django?

No universo da engenharia web do Django, **Template** é apenas o nome pra um arquivo `.html` completamente normal que conhecemos, mas que recebeu **superpoderes**. 

A diferença é que num HTML "morto/estático" você escreve manualmente no teclado "R$ 49,90" para o preço de um produto. Num *Template do Django*, o HTML consegue pausar o desenho na tela até receber as contas atualizadas e os preços do Banco de Dados gerados pelo código Python, usando furos e marcações (que chamamos de variáveis).

### 1.1 Onde os arquivos HTML devem morar?
O Django é extremamente focado em organização e exige que guardemos nossas coisas em gavetas perfeitamente padronizadas. 

Lembrando da aula passada, o nosso **aplicativo** principal do projeto se chama `core`. O Django nos pede que coloquemos os arquivos HTML deste app dentro de uma pasta chamada `templates` e, estranhamente, dentro dela criar *mais uma pastinha* com o nome do aplicativo. Vamos criar isso!

Certifique-se de que está dentro do servidor na pasta `meu_site` (com a bolha virtual `(venv)` do Python ativada no canto da tela) e digite no terminal:
```bash
mkdir -p core/templates/core
```

> **Por que essa repetição estranha de pastas "core/templates/core"?** 
> É uma das "Boas Práticas" padrão da indústria para trabalhar com aplicativos que crescem muito! Se lá no futuro a sua empresa criar um projeto gigantesco de site, como um Mercado Livre, e tiver 5 aplicativos diferentes rodando lá dentro e por azar cada um deles decidir ter a própria página principal com o nome `index.html`, o Cérebro do Django jamais vai desenhar a tela de pagamento do cliente na tela de contato, porque cada página de cada setor `index.html` estará bem segura isolada dentro de sua gavetinha final que leva o nome do próprio App!

---

## Passo 2: Criando o Seu Primeiro Molde HTML

Agora que as gavetas estão organizadas, vamos criar o papel de rascunho. 

Abra o arquivo HTML (se não existir, criaremos um do zero):
```bash
nano core/templates/core/index.html
```

Copie esse HTML de teste que eu criei, mas arregale os olhos para notar a "Mágica do Django", que chamamos de Tag de Variável sempre que usamos abraços de chaves duplas: `{{ ... }}`.

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Minha Primeira Página Dinâmica</title>
    <!-- Um estilo puro e simples importado rapidamente na alma do HTML, para agradar visualmente o cliente -->
    <style>
        body { font-family: 'Segoe UI', Arial, sans-serif; background-color: #2c3e50; text-align: center; padding: 50px; }
        .caixa { background: #ecf0f1; padding: 40px; border-radius: 12px; box-shadow: 0px 4px 15px rgba(0,0,0,0.3); display: inline-block; }
        h1 { color: #2980b9; }
        p { color: #34495e; font-size: 18px; }
        .poder { color: #e74c3c; font-weight: bold; font-size: 24px;}
    </style>
</head>
<body>

    <div class="caixa">
        <!-- OLHE PARA A VARIÁVEL ABAIXO SENDO CHAMADA -->
        <h1>Bem-vindo ao servidor, {{ nome_astronauta }}!</h1>
        
        <p>Você navegou até o servidor Amazon EC2 e é o visitante sorteado número <span class="poder">{{ acessos_totais }}</span> de hoje.</p>
        
        <p>A engenharia e inteligência executando os cálculos atrás desta tela foi escrita com foco em <strong>{{ linguagem_secreta }}</strong> para durar séculos na internet.</p>
    </div>

</body>
</html>
```
*(Para salvar e consolidar isso no editor `nano` do linux: aperte `Ctrl + O`, aperte `Enter`, depois feche apertando `Ctrl + X`).*

### O Que Aconteceu Lá em Cima?
Todo aquele texto que estiveja recheado de chaves e for desenhado como **`{{ chaveiro_duplo }}`** é considerado por nós desenvolvedores como uma **Variável de Template**. O navegador HTML não vai exibir `{{ nome_astronauta }}` de verdade como um texto, pelo contrário, ele vai colocar um bloco invisível aguardando desesperadamente o Cérebro superior do Python enviar a resposta definitiva dele pra tapar esses 3 buracos com informações fresquinhas! O que mostra que no futuro podemos colocar nomes de diferentes usuários de login nessas variáveis sem repetição de programação visual.

---

## Passo 3: Modificando a Lógica em Python (A View)

A tinta do CSS, o projeto das caixas e os moldes estão em pratos limpos. Mas e nosso colete salva-vidas? Alguém avisa o navegador que a mágica começou. A "View" em linguagem `.py`. 

Vamos abrir mais uma vez o arquivo onde mora o Cérebro de lógicas da Parte 1:
```bash
nano core/views.py
```
Apague a função que retornava lixo `HttpResponse` da aula passada e reconstrua a função inicial inteira de modo a ficar assim:

```python
# A função 'render' funciona como a tesoura, a cola e a fita: ela une e finaliza um Template HTML combinando com Dados de fundo dinâmicos
from django.shortcuts import render 

def pagina_inicial(request):

    # 1. Fase Brutal de Cálculos em Python
    # Na vida real, o Django conectaria no Amazon Aurora neste exato segundo, buscando a última fatura, quem logou, acessos e os produtos do cliente atual.
    visitante = "Viajante do Espaço"
    cliques_na_tela = 42
    tecnologia = "Python Moderno e Framework Web de Alta Performance"

    # 2. Empacotando essas variáveis num colete (O Empacotador de Presentes, Contexto / Dictionary)
    # A esquerda ('') fica EXATAMENTE IGUAL à variável solta sem sentido láctico que o Template chora esperando e à direita (=) injetamos e atracamos nossos cálculos que criamos aqui do nada agora acima! 
    pacote_de_dados = {
        'nome_astronauta': visitante,
        'acessos_totais': cliques_na_tela,
        'linguagem_secreta': tecnologia,
    }

    # 3. Final: O grande herói `render` une todo o motor: O pedido que foi tocado da rua (request), a caixa do Molde do pedreiro perfeitinho que acabamos de desenhar 'core/index.html' pelo renderizador do template, junto da caixa das variáveis com o pacote e cola em vida HTML real na tela!
    return render(request, 'core/index.html', pacote_de_dados)
```
Toda variação no tráfego de internet ou rotatividade de produtos na loja deve agora mudar única e exclusivamente pelas variáveis nesse colete, preservando inteiramente seu CSS valioso no Template!

---

## Passo 4: E as super Imagens que sobram ou CSS complexos e pesados? (Arquivos Estáticos)

A beleza arquitetural de um **Web-Site** pode requerer fotos de carros ou PDFs no site `logo-gigante-3d.png` e enormes estilos pesados em arquivos externos (`.css`). Para tudo o que não seja calculado, que **não mude sob hipótese nenhuma dinamicamente por cálculo e que não demande processamento do Cérebro Python**, o Django lida no método de **Arquivos Estáticos (Static Files)**.

É de semelhante natureza que lidamos as gavetas HTML, e mais uma maluca que ele pede:
Se você usar muito CSS para não amontoar tudo no HTML acima, crie e se organize numa pasta na frente de sua casinha estática (na mesma parte que criamos o da raiz do `manage.py` e fomos para dentro de `core`):
```bash
mkdir -p core/static/core
```

Aí basta subir, colocar seu site `.png` num cliente, com o `FTP/Github` que quiser e soltá-lo em `core/static/core/banner-carro.png`.

### Certo, eu puxei a estática pesada, como ligar ela na tela bonita visual ao invés do Python?
No topo absoluto que for no nível 1 do seu arquivo Molde Principal (`core/templates/core/index.html`), bem distante mas acima da própria Tag universal do `<!DOCTYPE html>`, você precisará conjurar a mágica de permissão e dizer que a Tela "Lê Estática", e essa tag diferente da variável dos dados por usar um sinal de porcenatgem `{% ... %}`:

```html
{% load static %}  <!-- <== Mágica invocada avisando para carregar pasta static para soltar recursos livres nela! -->
<!DOCTYPE html>
<html lang="pt-br">
<head>
   ...
   
   <!-- Em vez de usar aquele link da vida das pedras, use a rotação mágica dinâmica no css externo! -->
   <link rel="stylesheet" href="{% static 'core/meu-estilo-gigante.css' %}">
   
</head>
<body>
    
   <!-- Puxando o banner bruto do seu site da estática ao vivo usando template no src sem ter que memorizar URL de imagem! -->
   <img src="{% static 'core/banner-carro.png' %}" alt="Logo Gigante">

```
> **Por que focar no Django usar uma variável feia tag `{% static %}` debaixo do pano em uma simples chamada de `img_src`?**
> A genialidade do Backend Python entra em cena! O Cérebro calcula sozinho onde essa pasta Estática tá solta não importando que ele seja local do seu computador rodilho, porque assim que aquele dia milagroso chegar de você ligar o NGINX, subir num Amazon EC2 e subir no banco de dados S3 Buckets public nas nuvens ou trocar o domínio e estrutura original inteira, o motor conserta sozinho os links! Seu Frontend HTML, dessa forma, nunca saberá se você mudou os arquivos de máquina... Suas imagens antigas nunca explodem fora do site pela simples invocação estática no Django!

---

---

## Passo 5: O Grande Gran Finale (Exibindo Produtos do Banco com Laços de Repetição)

Para coroar o ciclo do Django, vamos pegar aquela tabela de Produtos que desenhamos no Admin na aula anterior e forçar o HTML a gerar repetidamente caixas de produtos na tela sem precisarmos copiar e colar linhas! Para isso usaremos a Tag de Repetição **`{% for %}`**, o equivalente ao Looping do Python.

### 5.1 O Código Completo do `core/views.py`
A sua View agora vai pescar no banco de dados, além das variáveis antigas, também a tabela inteira do estoque e enfiar no "colete".

Abra novamente `nano core/views.py` e apague tudo que tinha lá. Deixe esse arquivo **exatamente e inteiramente** assim:

```python
from django.shortcuts import render
from .models import Produto  # <== Importação vitalícia da tabela do Banco!

def pagina_inicial(request):

    # 1. Fase Brutal de Cálculos em Python
    visitante = "Viajante do Espaço"
    cliques_na_tela = 42
    tecnologia = "Python Moderno e Framework Web de Alta Performance"
    
    # 2. Indo no banco e trazendo todos os cadastros
    produtos_do_banco = Produto.objects.all()

    # 3. Empacotando absolutamente tudo num "colete" Dictionary
    pacote_de_dados = {
        'nome_astronauta': visitante,
        'acessos_totais': cliques_na_tela,
        'linguagem_secreta': tecnologia,
        'lista_de_produtos': produtos_do_banco, # <== O Banco de Dados adicionado aqui!
    }

    # 4. Final: Despachando
    return render(request, 'core/index.html', pacote_de_dados)
```

### 5.2 O Código Completo do HTML (`core/templates/core/index.html`)

No HTML, vamos "desempacotar" a variável de lista usando o `{% for %}`. O Django vai gerar dezenas de cópias de blocos de HTML até esgotar todos os produtos que achar sem você transpirar escrevendo. Também usaremos a marcação `{% if %}` para checar se ele está em estoque, ficando verde se verdadeiro e vermelho se falso!

Abra o arquivo `nano core/templates/core/index.html` e apague tudo. Deixe ele **exatamente** assim, inteiro:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Minha Primeira Página Dinâmica e Estocada</title>
    <!-- CSS completo embutido, sem partes omitidas -->
    <style>
        body { font-family: 'Segoe UI', Arial, sans-serif; background-color: #2c3e50; text-align: center; padding: 50px; }
        .caixa { background: #ecf0f1; padding: 40px; border-radius: 12px; box-shadow: 0px 4px 15px rgba(0,0,0,0.3); display: inline-block; max-width: 800px; text-align: left; }
        h1 { color: #2980b9; text-align: center;}
        p { color: #34495e; font-size: 18px; text-align: center;}
        .poder { color: #e74c3c; font-weight: bold; font-size: 24px;}
        .vitrine { margin-top: 30px; border-top: 3px solid #bdc3c7; padding-top: 20px;}
        .produto-card { background: white; border-left: 5px solid #27ae60; padding: 15px; margin-bottom: 15px; border-radius: 4px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .preco { font-size: 22px; color: #27ae60; font-weight: bold; }
        .esgotado { border-left-color: #e74c3c; }
        .esgotado .preco { color: #e74c3c; }
        .status { font-weight: bold; margin-top: 10px; }
    </style>
</head>
<body>

    <div class="caixa">
        <h1>Bem-vindo ao servidor, {{ nome_astronauta }}!</h1>
        <p>Você é o visitante sorteado número <span class="poder">{{ acessos_totais }}</span> de hoje.</p>
        <p>Escrito com foco em <strong>{{ linguagem_secreta }}</strong>.</p>
        
        <!-- INÍCIO DA MÁGICA DE REPETIÇÃO DO ESTOQUE -->
        <div class="vitrine">
            <h2>Nossa Loja (Dados Direto do Admin!):</h2>
            
            {% for produto in lista_de_produtos %}
                
                <!-- Verificando a condicional de Estoque -->
                {% if produto.em_estoque %}
                <div class="produto-card">
                    <h3 style="margin: 0; color: #2c3e50;">{{ produto.nome }}</h3>
                    <div class="preco">R$ {{ produto.preco }}</div>
                    <div class="status" style="color: #27ae60;">✅ Disponível para compra</div>
                </div>
                {% else %}
                <div class="produto-card esgotado">
                    <h3 style="margin: 0; color: #2c3e50;">{{ produto.nome }}</h3>
                    <div class="preco">R$ {{ produto.preco }}</div>
                    <div class="status" style="color: #e74c3c;">❌ Esgotado no momento</div>
                </div>
                {% endif %}

            {% endfor %}
            <!-- FIM DO LOOP -->
        </div>

    </div>

</body>
</html>
```

Agora salve e dê "Refresh" na sua aba do seu site rodando (`http://SEU_IP:8000/`)! Cada item verde que foi ou for salvo lá no painel Admin pula direto pra tela desenhada em HTML num ciclo incrivelmente coeso!

---

## Repassando a Partida, A Máquina Gira Agora!

Para finalizar tudo sendo um verdadeiro arquiteto Backend Django:
1. O Cliente clica pelo telefone **Pede na Rua (`http://O_IP/`)**.
2. **O Carteiro Escuta As Rotas (`urls.py`):** "Endereço principal da rua central, bate pra casinha do `pagina_inicial`!".
3. **O Motor De Processamento Gira (`views.py`):** "Bora, criem pacotes que mandamos agora! Nome, visitas e varrer o banco Aurora AWS em `Produto.objects.all()`, façam o contexto! Vamos entregar esse Dictionary Context para a `render` e juntar esses dados tudo com os pedreiros da HTML Visual".
4. **Acabou a Obra! (O Resultado Final Template Substituído):** O Molde puxa a luz e entende "Legal do `render` e as Variáveis! Toma o pacote! Limpar furos e girar um `for` pra desenhar botões repetidos da lista de estoque!". E assim a página brilhante injetada lá visualmente nasce sem dores para os seus clientes de forma orgânica e flexível.

A engenharia do seu Back-end começa a brilhar! O site em sua AWS tem o sangue puro rodando dentro com o Banco de dados perfeitamente pronto aguardando mais views conectadas na web. Essa mágica do Padrão MTV (Model, Template, View) resolve os maiores problemas do desenvolvimento de Software unindo segurança e perfeição. Aproveite os Template Tags sem limites!
