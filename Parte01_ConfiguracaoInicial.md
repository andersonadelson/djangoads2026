# Tutorial Completo: Introdução ao Django no Amazon Linux 2023

Olá! Bem-vindo ao mundo do desenvolvimento de sistemas web com Python.

Para que este guia seja fácil para leigos, vamos devagar. O **Django** é um "framework". Pense nele como uma caixa cheia de ferramentas e moldes já prontos que nos ajudam a construir sites seguros, rápidos e modernos, sem precisar reinventar a roda.

Neste tutorial, vamos configurar um servidor limpo com **Amazon Linux 2023**, instalar o Django e criar a sua primeira página web, explicando o que significa cada comando.

---

## Passo 1: Preparando o Terreno (Amazon Linux 2023)

O Amazon Linux 2023 usa um gerenciador de pacotes chamado `dnf`. Pense no `dnf` como a "loja de aplicativos" oficial do seu servidor pelo terminal. Vamos usá-lo para instalar as coisas que precisamos.

### 1.1 Atualizar o sistema

Sempre que começamos um projeto, a primeira regra de ouro: atualizar o servidor. Digite no terminal do seu servidor:

```bash
sudo dnf update -y
```

_(O `sudo` significa "fazer como administrador chefe", `dnf` é a nossa loja de aplicativos, `update` significa atualizar, e `-y` diz "sim" para qualquer pergunta automática)._

### 1.2 Instalar o Python

O Django é escrito na linguagem **Python**, então o servidor precisa entender essa língua. Vamos instalar o Python e uma ferramenta chamada `pip`, que é a "loja de aplicativos exclusiva do Python":

```bash
sudo dnf install python3 python3-pip -y
```

---

## Passo 2: A Regra de Ouro (Ambientes Virtuais)

**Melhor Prática:** Nunca instale o Django diretamente solto no sistema operacional inteiro. Nós usamos o que chamamos de "Ambiente Virtual" (Virtual Environment).

Imagine um ambiente virtual como uma "bolha de vidro" fechada onde seu projeto vive. Tudo o que instalamos (como o Django) fica dentro dessa bolha e não bagunça os outros programas do servidor. Você pode ter vários projetos no futuro, cada um em sua bolha isolada.

### 2.1 Criar a pasta do seu site

Vamos criar uma pasta para abrigar o nosso projeto e "entrar" nessa pasta:

```bash
mkdir meu_site
cd meu_site
```

### 2.2 Criar a bolha (Ambiente Virtual)

Agora, vamos criar o ambiente virtual. Vamos chamá-lo de `venv` (virtual environment):

```bash
python3 -m venv venv
```

### 2.3 Entrar na bolha (Ativar o Ambiente Virtual)

Toda vez que você for programar neste projeto, você precisa "entrar na bolha". Para ativá-la, digite:

```bash
source venv/bin/activate
```

**Como saber se deu certo?** Olhe para o seu terminal. Você verá que apareceu um `(venv)` bem no início da linha de texto. Isso significa que você está dentro da bolha de forma segura!

_(Nota: Quando você encerrar o seu dia de trabalho e quiser sair da bolha, basta digitar `deactivate`. Mas **não o faça agora**, pois precisamos continuar dentro dela para os próximos passos)._

---

## Passo 3: Instalando o Django

Agora que estamos isolados dentro do `(venv)`, vamos chamar nossa loja de aplicativos do Python (`pip`) para baixar o Django:

```bash
pip install django
```

---

## Passo 4: Iniciando o Seu Projeto

O Django divide suas coisas em "Projetos" e "Aplicativos (Apps)":

- **Projeto (Project):** É o cérebro inteiro do seu site (as configurações gerais de banco de dados, idioma, fuso horário, senhas).
- **Aplicativo (App):** São pedaços independentes do site (por exemplo, um app de "Carrinho de Compras", um app de "Blog", um app de "Login de Usuário"). Um projeto é formado por vários aplicativos.

### 4.1 Criando o cérebro (O Projeto)

Ainda dentro da pasta `meu_site`, crie o projeto que vamos chamar de `meu_projeto`:

```bash
django-admin startproject meu_projeto .
```

_(**Atenção:** Repare que tem um ponto final `.` depois de `meu_projeto`. Ele é essencial! Ele diz ao Django para construir as coisas bem na pasta atual que estamos, sem criar um monte de subpastas desnecessárias)._

### 4.2 Explorando as pastas

Se você digitar `ls` (listar arquivos) agora, verá algo muito especial que acabou de surgir:

- **Uma pasta `meu_projeto`:** Onde ficam as configurações centrais.
- **Um arquivo `manage.py`:** A partir de hoje, esse arquivo será o seu "mordomo". Você vai usá-lo para comandar o Django para fazer quase tudo.

---

## Passo 5: Criando a Primeira Funcionalidade (App)

Vamos criar um pedaço do nosso site (App), que será responsável por mostrar a nossa página inicial. Chamaremos esse aplicativo de `core` (que significa "núcleo" ou "coração" do sistema).

```bash
python3 manage.py startapp core
```

### 5.1 Avisar o Cérebro sobre esse App novo

O Projeto não sabe automaticamente que o nosso App `core` existe. Precisamos apresentá-los.

Abra o arquivo de configurações central: `meu_projeto/settings.py`. Você pode usar o editor de texto chamado `nano` direto no terminal:

```bash
nano meu_projeto/settings.py
```

Desça a tela até a parte escrita **`INSTALLED_APPS`**. Vai parecer uma lista. Adicione o nosso `'core',` lá no final:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'core',  # <-- Adicione exatamente esta linha! (Não esqueça a vírgula)
]
```

_(Para salvar no editor nano: aperte `Ctrl + O`, depois clique `Enter`, depois feche apertando `Ctrl + X`)._

---

## Passo 6: Como as Páginas Funcionam no Django (Views e Rotas)

O fluxo da internet no Django segue 3 passos básicos sempre que alguém clica num link:

1. **O Endereço (URL):** O usuário digita um link mágico (ex: `seusite.com/sobre-nos`).
2. **A Rota (urls.py):** O Django diz: "Recebi um chamado no `/sobre-nos`. Quem cuida disso?".
3. **A Exibição (View):** A página que vai calcular tudo o que deve voltar e entregar o HTML final pro usuário ver.

### 6.1 Criando a tela final (A View)

Vamos criar a nossa primeira página web real. Abra o arquivo que lida com a lógica: `core/views.py`:

```bash
nano core/views.py
```

E escreva dentro dele:

```python
from django.shortcuts import render
from django.http import HttpResponse

# Esta é a nossa primeira página (uma View)
def pagina_inicial(request):
    return HttpResponse("<h1>Olá, Mundo! Meu primeiro projeto em Django está vivo no Amazon Linux!</h1>")
```

Salvou? Tudo que essa página faz é pegar um pedido (`request`) de acesso à internet e devolver (`return`) um mega texto gigante formatado em HTML (é pra isso que serve o `<h1>`).

### 6.2 Criando o mapa (A Rota / URL)

Agora o Cérebro precisa saber que quando alguém entrar na raiz do site (sem digitar nada depois do ".com"), essa vista deve ser chamada.

Abra as rotas mestre do site em `meu_projeto/urls.py`:

```bash
nano meu_projeto/urls.py
```

Deixe o arquivo exatamente assim (exclua os comentários em inglês no topo, se quiser):

```python
from django.contrib import admin
from django.urls import path
from core import views  # <-- 1. Importante: "Avisar" este arquivo que nossas Views estão disponíveis

urlpatterns = [
    path('admin/', admin.site.urls),  # Este já vem pronto (é o painel de adm secreto)
    path('', views.pagina_inicial, name='home'), # <-- 2. Dizemos: Se o caminho estiver em branco (''), use a pagina_inicial do core
]
```

---

## Passo 7: Rodando o Site na Nuvem da Amazon

Como não estamos trabalhando em um notebook na mesa de casa, mas num servidor nas nuvens (Amazon EC2), temos um último passo de segurança. O Django é paranóico e bloqueia por padrão acessos de "bombeiros" externos. Temos que liberar.

### 7.1 Liberar visitas externas (ALLOWED_HOSTS)

Ainda editando configurações, abra com o nano o seu `meu_projeto/settings.py` novamente:

```bash
nano meu_projeto/settings.py
```

Procure e deixe a linha `ALLOWED_HOSTS` assim:

```python
ALLOWED_HOSTS = ['*']  # O asterisco significa "liberado para todos acessarem por qualquer IP por enquanto."
```

### 7.2 Preparar o Banco de Dados Central

O Django é inteligente e já vem com um painel de administrador e controle de logins integrados. Mas, para isso funcionar, precisamos "selar" o banco de dados inicial dele (Migrações).
Execute esse comando, sempre pedindo pro mordomo:

```bash
python3 manage.py migrate
```

_(Você verá vários "OKs" verdes descendo na tela)._

### 7.3 Criar o Usuário Administrador

Para acessar o painel de administrador que já vem integrado ao Django, você precisa criar uma conta com superpoderes (um superusuário):

```bash
python3 manage.py createsuperuser
```

O terminal vai pedir que você defina um Nome de Usuário (Username), um E-mail (Email address) e uma Senha (Password). **Nota:** no Linux, quando você digita a senha, os caracteres não aparecem na tela por motivos de segurança. Apenas digite sem ver e aperte Enter no final.

### 7.4 Ligar a "Chave de Luz" (O Servidor)

É o grande momento. Vamos levantar o servidor publicamente pedindo para ele "ouvir" abertamente (0.0.0.0) na porta especial 8000:

```bash
python3 manage.py runserver 0.0.0.0:8000
```

Se não der nenhum erro, seu terminal ficará "congelado" esperando visitas. O seu servidor está no ar!

Ou use o comando para a porta 80

```bash
sudo /home/ec2-user/environment/meu_site/venv/bin/python3 manage.py runserver 0.0.0.0:80
```

---

## Passo 8: Como ver o site de verdade?

1. Entre no painel da sua **Amazon Web Services (AWS)**.
2. Na sua tela de instâncias EC2 (o seu servidor Linux), procure por uma informação chamada **"Public IPv4 Address"** ou **Endereço IPv4 Público**. É uma sequência de números como `15.111.23.442`.
3. Abra a aba do seu navegador de internet do seu computador e digite: `http://SEU_IP_AQUI:8000`

> **UM AVISO LEIGO:** Se demorar infinito para carregar a página e não der erro nenhum... **A culpa não é do Django!** É a barreira de proteção de fábrica da Amazon.
> Você precisa entrar nas configurações da AWS (Procure por **Security Groups** conectado à instância) -> ir em "Inbound Rules" (Regras de Entrada) -> Cadastrar uma nova regra Custom TCP para a **porta 8000** permitindo qualquer lugar (0.0.0.0/0). A página carregará em um milissegundo depois disso.

## Resumo dos Comandos Que Você Aprendeu

- `python3 -m venv venv` ➔ **Cria a bolha de segurança**
- `source venv/bin/activate` ➔ **Entra na bolha**
- `pip install django` ➔ **Baixa a magia toda**
- `django-admin startproject NOME_DO_MEU_SITE .` ➔ **Cria a fundação e cérebro do site**
- `python3 manage.py startapp NOME_DO_PEDACO` ➔ **Cria um cômodo (Aplicativo) novo na casa**
- `python3 manage.py migrate` ➔ **Prepara as tabelas cimentadas no banco de dados**
- `python3 manage.py createsuperuser` ➔ **Cria o usuário e senha para acessar o painel admin**
- `python3 manage.py runserver 0.0.0.0:8000` ➔ **Coloca o site no ar**

A partir de agora, para aprender mais, o próximo o passo seriam os **Templates** (Deixar as nossas `Views` retornarem arquivos `.html` bonitos no lugar daquele texto feio de <h1>) e **Modelos** (Como salvar dados em banco de dados). Mas as bases da sua máquina Cloud estão impecavelmente construídas! Parabéns!
