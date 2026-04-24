# Tutorial Completo (Parte 3): Consumindo uma API Externa (PokéAPI) no Django

Seja muito bem-vindo a uma das áreas mais emocionantes e lucrativas da programação web moderna: fazer o seu site conversar com outros servidores pelo globo para puxar dados valiosos ao vivo! 

Hoje vamos construir uma verdadeira aplicação. Nosso Amazon EC2 vai se transformar numa **Pokédex** que consome dados diretos dos servidores oficiais do famoso mundo de Pokémon (através da PokéAPI) e exibe as fotos na tela do cliente.

---

## Passo 1: Mas afinal, o que é uma API?

**API** (Application Programming Interface / Interface de Programação de Aplicação) é como o "Garçom" interligando as cozinhas da internet. Imagine que você tem um restaurante, e por ser muito difícil caçar um peixe e fazer os cortes do salmão, você faz uma ligação pra uma empresa e ela manda a "marmita pronta" de peixes pro seu balcão.  
O seu site no painel da Amazon faz um simples "pedido" para o Grande Servidor de Pokémons, e eles mandam uma marmita digital (um monte de texto padronizado chamado arquivo **JSON**) recheada dos pesos, foto, nome e vida. Nosso único trabalho no Python é pegar o que interessa e jogar no molde visual HTML do Django!

---

## Passo 2: Instalando o "Telefone" (Biblioteca Requests)

Para que o nosso Cérebro Python consiga realizar chamadas na rua (e "telefonar" para a API externa do Pokémon de forma muito fácil e simples), nós usamos a biblioteca mais amada e clássica de todas, chamada `requests`.

Volte ao terminal sombrio do seu Amazon EC2 (certifique-se de estar com a mágica bolha virtual `(venv)` do Python sempre ativada no cantinho!) e instale o pacote de chamador telefônico na internet:
```bash
pip install requests
```

---

## Passo 3: O Cérebro Python Ligando para a API (A View)

A lógica do fluxo nunca muda. O usuário avisa a **Rota** (A URL) que clicou em visitar, e a URL acorda O Cérebro da **View** (Nossa função de gerar a tela final).

Vamos avisá-lo que antes de montar tudo em tela e colorir as imagens no Template que nós desenhamos antigamente, ele **precisa** ir na rua puxar as estáticas pro Contexto! 

Abra o arquivo onde mora nossa engenharia forte:
```bash
nano core/views.py
```

Destrua a função anterior da aula e mude de vez a lógica do nosso painel substituindo tudo por este lindo script:

```python
# =========================================================================
# ATENÇÃO MÁXIMA: NUNCA ESQUEÇA A LINHA 'import requests' AQUÍ EM CIMA!
# Sem ela o seu código vai dar o erro "NameError: requests is not defined"
# =========================================================================
from django.shortcuts import render
import requests  # <-- Trazendo pra dentro O Telefone pra ligar pra Pokéapi na rua

def pagina_inicial(request):

    # 1. ESCOLHA O QUE QUER BUSCAR E LIGUE O RADAR!
    # Vamos pedir na URL sagrada da API todos os dados soltos e bagunçados envolvendo o Pikachu
    url_da_api = "https://pokeapi.co/api/v2/pokemon/pikachu"
    
    # 2. FAZENDO A CHAMADA MÁGICA EXTERNA
    # Nesse linha ele trava e vai na internet buscar. O garçom atende o pedido e a variável 'resposta' fica recheada
    resposta = requests.get(url_da_api)

    # 3. TRADUZINDO A RESPOSTA CONFUSA (DO JSON CRU) PARA O DOCE PYTHON
    # A resposta de lá vem em texto duro cru de leitura pesada (Um Livro inteiro), fazemos virar caixinhas de dados Dicionário!
    dados_do_pokemon = resposta.json()

    # 4. A MINERAÇÃO DE OURO (SÓ O QUE IMPORTA NESSA MARMITA ENORME)
    nome = dados_do_pokemon['name']  # Pinça fora só o nome
    peso = dados_do_pokemon['weight'] # Pinça só o peso
    # A url da foto frontal padrão de internet tá bem escondida numa sub-caixa dentro do armário deles, nós vamos lá pegar o atalho principal pra não ter que baixar!
    imagem = dados_do_pokemon['sprites']['front_default'] 

    # 5. NOSSA PREPARAÇÃO DE PRESENTE PRO HTML FINAL DA TELA DO CLIENTE (O Contexto)
    # Empacotamos como aprendemos na aula 2! A esquerda: para os 'Buracos do HTML', a direita: pra lixeira viva que acabamos de roubar/calcular!
    pacote_de_dados = {
        'nome_pokemon': nome.capitalize(),  # Já entrega pro cliente a Inicial Da Letra toda Maiúscula caprichada! (Pikachu)
        'peso_pokemon': peso,
        'imagem_pokemon': imagem,
    }

    # 6. ENVIAR PARA A PÁGINA COLAR E FINALIZAR A OBRA NA NUVEM!
    return render(request, 'core/index.html', pacote_de_dados)
```

*(Lembre-se do nosso atalho para sair do Linux: Salve e confirme `Ctrl+O`, `Enter` e corte a conexão do editor `Ctrl+X`)*.

---

## Passo 4: Desenhando o Pokémon na Tela (O Template HTML)

Para a Glória do Django ser unida à web mundial, O Cérebro ligou na rua, obteve os dados e disparou ao Template HTML nossas caixinhas cruas preenchidas `'nome_pokemon'` e `'imagem_pokemon'`.

Abra o construtor HTML que a View chamou para o Molde da Pokedéx!
```bash
nano core/templates/core/index.html
```

Delete totalmente o HTML da aula passada do Astronauta Mestre, e coloque nosso código pra desenhar esse brinquedo fabuloso e vermelho com nossa **Mágica Template do Django `{{ }}`**!

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Pokedex Viva do Django Cloud</title>
    <!-- CSS super bacana que deixará parecendo uma Pokedex original Nintendo! -->
    <style>
        body { font-family: 'Arial', sans-serif; background-color: #3b4cca; text-align: center; padding: 50px; }
        .pokedex-estrutura { 
            background: #ff0000; padding: 30px; border-radius: 15px; 
            box-shadow: 0px 10px 20px rgba(0,0,0,0.5); 
            display: inline-block; width: 320px;
            border: 4px solid #cc0000;
        }
        .telinha-digital { 
            background: #f5f5f5; border: 4px solid #222; 
            border-radius: 10px; padding: 25px; text-align: center;
        }
        h1 { color: #333; margin: 0; font-size: 32px; letter-spacing: 2px;}
        img { width: 150px; margin-top: 10px; filter: drop-shadow(0 8px 4px rgba(0,0,0,0.2)); }
        p { color: #5B5B5B; font-size: 18px; font-weight: bold; background: #FFCB05; border-radius: 6px; padding:4px;}
        
        .bolinha-azul { width: 40px; height: 40px; background: #007bff; border-radius: 50%; border: 3px solid #fff; box-shadow: 0px 0px 10px rgba(0,0,0,0.8); margin-bottom: 20px;}
    </style>
</head>
<body>

    <div class="pokedex-estrutura">
        <div class="bolinha-azul"></div>
        <div class="telinha-digital">
        
            <!-- E AGORA, O RECHEIO BRUTAL VINDO DA API ESTOICAMENTE INJETADO DA RUA INTERNET! -->
            <h1>{{ nome_pokemon }}</h1>
            
            <!-- Atenção aqui: não pomos o link nós mesmos, ele põe direto do url do contexto que veio dos estados unidos pelo cérebro da view da amazon na src = "" !! -->
            <img src="{{ imagem_pokemon }}" alt="Foto Fresca da Base de Dados da API">
            
            <p>Peso Médio: {{ peso_pokemon }}00 gramas</p>
            
        </div>
    </div>

</body>
</html>
```

*(Salve e feche `nano` usando os mesmos comandos Linux)*

---

## Passo 5: Atualize o celular e veja a Mágica Internacional Subir Da Amazon!

Para finalizar como engenheiro:
1. Certifique-se de não ter parado o tráfego do servidor que devia estar rodando estanque lá no terminal e piscando (`python3 manage.py runserver 0.0.0.0:8000`).
2. Acesse seu Servidor do Public IP da nuvem da Amazon pelo navegador de lá, apertando `"Recarregar Tela"` no mesmo local antigo daquele Painel do `Passo 2`.

### O Que Acabou Ocorrer pelo Globo Terrestre Em Três Milésimos De Segundo Quando Desenhada?
Quando você clicou "F5 (Refrescar)" na rota `/`:
1. Seu Django identificou e gritou: *"Opa, bateram na nossa requisição de Rota, passem pro cérebro da VIEW `pagina_inicial`!"*
2. A View no Amazon EC2 atendeu, mas disse no trâmite: *"Antes de finalizar, me dê uma licença meu cliente! Eu preciso pegar o pacote requests e telefonar pra central mundial hospedada na PokéAPI pra descobrir o Pikachu em formato JSON!"*
3. O servidor longe atendeu o Brasil, enviando uma resposta massiva via Internet de tudo que o Pokemon era!
4. O `Python` filtrou em uma fração de piscar as "sujeiras do JSON que importava na marmita" pro nome, foto frontal original real estática do servidor deles e peso. Assinada à Variável `Context`, colou esses três buraquinhos cruciais no painel da HTML e desenhou ela. 
5. Seus olhos viram uma Pokedéx de vidro linda surgir sem que seu Amazon Linux soubesse jamais criar as fotos localmente ou desenhar esse pokemon, tudo por ser o Intermediador de Dados!

Essa é a grande essência majestosa do **"Backend".** Imagine o poder imensurável de daqui para a frente trocar "Pokémon.co/Pikachu" pela fantástica URL da API oficial da **Cotação de Bitcoin Mundial ao vivo do dia**, o recebimento de pagamento do **PayPal/MercadoPago**, ou a API do **Clima/Previsão do Tempo** em São Paulo do `ClimaTempo` ao usuário acessar sua página Inicial sem ver esforço algum seu. A ponte magnética da Web inteira agora obedece às lógicas que você manda ela escrever pelo Cérebro para entregar mastigada no HTML! A glória de desenvolvimento Web nas nuvens!
