# Tutorial Completo (Parte 5): Integrando um Framework CSS Profissional

Agora que você já domina o Padrão MTV do Django, lida com uploads de imagens e faz buscas complexas no banco de dados, chegou a hora de dar **cara de Amazon, de Mercado Livre ou de Magazine Luiza** ao seu humilde projeto.

Para não precisar perder dias da sua vida desenhando CSS do absoluto zero (tamanhos, sombras, alinhamentos, botões responsivos pro celular), a indústria utiliza os **Frameworks CSS**. O mais famoso, fácil e poderoso deles (e que o mundo inteiro usa de graça) é o **Bootstrap 5**, criado pelo Twitter.

---

## Passo 1: O Segredo do CDN (Content Delivery Network)

Para colocar o projeto de estilos da empresa Bootstrap dentro do seu servidor sem precisar sequer baixar nenhum arquivo CSS pesado para sua máquina AWS local, você usará tecnologia **CDN**. Trata-se de um simples `<link>` mágico de internet injetado no cabeçalho do `index.html`. O navegador do seu cliente fará o download do estilo direto dos gigantescos servidores do Bootstrap na nuvem, ignorando completamente sua AWS para baixar tinta, acelerando a página em 400%.

Abra novamente o seu HTML (`nano core/templates/core/index.html`) e **apague TUDO**. Esqueça as bordas toscas que programamos antigamente.

Deixe seu código HTML **Exatamente Inteiro Assim**, copiando do topo da tag `<!DOCTYPE>` até o encerramento puro `</html>`:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- Permite abrir perfeitamente num Celular -->
    <title>Minha Vitrine Profissional Modernizada</title>
    
    <!-- MÁGICA DO FRAMEWORK: O Link do Bootstrap 5 (CDN) conectando ao site do Bootstrap -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Adicionando uma biblioteca GIGANTE de ícones super bonitos gratuitos -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css">
    
    <style>
        /* CSS mínimo do seu gosto para consertar o que a biblioteca não tiver ou não conseguir */
        .card-personalizado { transition: transform 0.2s, box-shadow 0.2s; }
        .card-personalizado:hover { transform: translateY(-5px); box-shadow: 0 10px 20px rgba(0,0,0,0.15) !important; }
    </style>
</head>
<body class="bg-light"> <!-- bg-light = "Background Light", ou seja, avisa pro Bootstrap pintar o fundo da tela de cinza super clarinho! -->

    <!-- HEADER / BARRA DE NAVEGAÇÃO SUPERIOR RESPONSIVA -->
    <!-- navbar-dark bg-dark pinta a barra de uma preta muito chique -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-5 shadow">
        <div class="container">
            <a class="navbar-brand fw-bold" href="/"><i class="bi bi-shop me-2"></i> Lojinha Amazon EC2</a>
        </div>
    </nav>

    <!-- CONTAINER PRINCIPAL CENTRALIZADO -->
    <!-- (container diz: Não espalhe meu conteúdo nas bordas absurdas extremas da tela de TV da sala da minha casa, concentre as vitrines certas no meio!) -->
    <div class="container">
        
        <!-- CABEÇALHO E MENSAGEM DO SEU PYTHON ENVOLTO NUM GRID -->
        <div class="row mb-5 text-center">
            <div class="col-12">
                <h1 class="display-4 fw-bold text-primary">Bem-vindo, {{ nome_astronauta }}!</h1>
                <p class="lead text-muted">A inteligência atrás dos cálculos desta tela está escrita com <strong>{{ linguagem_secreta }}</strong>.</p>
                <div class="badge bg-warning text-dark fs-6 mt-2">Visitas à Página: {{ acessos_totais }}</div>
            </div>
        </div>

        <!-- BARRA DE BUSCA EM MODO CARD PROFISSIONAL -->
        <div class="row justify-content-center mb-5">
            <div class="col-md-8 col-lg-6">
                <!-- .card .shadow-sm significa Cartão com uma Sombrinha pequena para sobressair chique no design Flat Moderno! -->
                <div class="card shadow-sm border-0 rounded-4">
                    <div class="card-body p-4">
                        <form method="GET" class="d-flex gap-2">
                            <input type="text" name="busca_do_usuario" class="form-control form-control-lg" placeholder="Busque nossos produtos pelo nome..." value="{{ palavra_pesquisada|default_if_none:'' }}">
                            <button type="submit" class="btn btn-primary btn-lg px-4"><i class="bi bi-search"></i></button>
                        </form>
                    </div>
                </div>
            </div>
        </div>

        <!-- LISTA DIVISORA DO ESTOQUE -->
        <h3 class="mb-4 text-secondary border-bottom pb-2"><i class="bi bi-box-seam me-2"></i> Nosso Estoque Atual</h3>
        
        <!-- ============================================== -->
        <!-- INCIO O BENDITO FOR E O GRID AUTOMÁTICO -->
        <!-- ============================================== -->
        <div class="row g-4"> <!-- O 'g-4' aplica os Gap, Goteiras automáticas e super bonitas entre os cards na grade-->

            {% if lista_de_produtos %}
                {% for produto in lista_de_produtos %}
                
                <!-- A MÁGICA RESPONSIVA (col-12 no smartphone, col-md-6 no tablet, col-lg-4 no notebook da empresa). Se a tela entorta, ele cai quebrando como mágica! -->
                <div class="col-12 col-md-6 col-lg-4"> 
                    <!-- h-100 garante as colunas do mesmo tamanho pra não ter layout quebrado de foto baixa e outra alta -->
                    <div class="card h-100 shadow-sm border-0 card-personalizado overflow-hidden rounded-4">
                        
                        <!-- Verificação de Imagem vinda do AWS (Se não houver foto, gera um quadrado cinza substituto) -->
                        {% if produto.imagem %}
                            <img src="{{ produto.imagem.url }}" class="card-img-top w-100" alt="{{ produto.nome }}" style="height: 250px; object-fit: cover;">
                        {% else %}
                            <div class="bg-secondary text-white d-flex flex-column align-items-center justify-content-center" style="height: 250px;">
                                <i class="bi bi-image" style="font-size: 3.5rem; opacity: 0.5;"></i>
                                <span class="opacity-50 mt-2">Sem Imagem</span>
                            </div>
                        {% endif %}

                        <div class="card-body text-center d-flex flex-column">
                            <h5 class="card-title fw-bold text-dark">{{ produto.nome }}</h5>
                            <!-- O mt-auto força o preço pro fundo do card grudado no rodape bonito! -->
                            <h3 class="text-success fw-bold my-4 mt-auto">R$ {{ produto.preco }}</h3>
                            
                            <!-- Botões/Badges Verdes e Vermelhos nativos usando a semântica nativa "success" e "danger" do Bootstrap -->
                            {% if produto.em_estoque %}
                                <span class="badge bg-success py-2 px-3 fs-6 w-100 rounded-pill"><i class="bi bi-check-circle me-1"></i> Disponível na Loja</span>
                            {% else %}
                                <span class="badge bg-danger py-2 px-3 fs-6 w-100 rounded-pill"><i class="bi bi-x-circle me-1"></i> Esgotado no Momento</span>
                            {% endif %}
                        </div>

                    </div>
                </div>
                {% endfor %}
            {% else %}
                <!-- Mensagem de erro em tamanho colossal caso a busca de nome no models.py não der nada -->
                <div class="col-12 text-center py-5">
                    <i class="bi bi-emoji-frown text-danger" style="font-size: 5rem;"></i>
                    <h2 class="mt-4 text-secondary">Nenhum produto foi encontrado.</h2>
                    <p class="text-muted fs-5">Lamentamos muito, não encontramos nada no estoque contendo "<strong>{{ palavra_pesquisada }}</strong>".</p>
                    <a href="/" class="btn btn-outline-primary btn-lg mt-3 rounded-pill px-5"><i class="bi bi-arrow-left me-2"></i> Limpar Filtro e Mostrar Tudo</a>
                </div>
            {% endif %}

        </div> <!-- Fim da Row Principal -->

    </div> <!-- Fim do Container -->

    <!-- RODAPÉ CLÁSSICO DE LOJAS -->
    <footer class="bg-dark text-white text-center py-4 mt-5">
        <div class="container">
            <p class="mb-0 text-white-50"><i class="bi bi-suit-heart-fill text-danger me-1"></i> Criado de forma sensacional por um dev Django. © 2024 Lojinha AWS.</p>
        </div>
    </footer>

    <!-- Acessórios de JS (Não mude, ajuda os botões secretos do Bootstrap a rolarem, e dropdowns dos ícones funcionarem por conta própria) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

## Passo 2: Assista o Milagre do Design

Salve este glorioso e mastigado artefato CSS e atualize o navegador que exibe a porta `8000` em sua casa!

Você vai presenciar a maior transformação que poucos comandos geram. Sem utilizar frameworks React, e sem possuir dias mexendo em arquivos `.css` que quebram à toa, você acaba de invocar uma verdadeira plataforma comercial construída inteira com Cartões Responsivos, Layouts em Grid (Grade flexível pro celular), Sombras bonitas `shadow-sm`, Imagens devidamente aparadas para caber `object-fit:cover` e Botões de Ação limpos redondinhos `rounded-pill`. Tudo atrelado e operando nas informações e fotos embutidas na alma do Python.

Este é o poder destrancado do Full Stack — Uma interface luxuosa se ligando à nuvem da AWS via Painel Administrativo.
