# Tutorial Completo (Parte 3): Uploads e Otimização Automática de Imagens

Quando começamos a lidar com imagens reais enviadas por usuários ou reabastecendo o estoque da loja no Admin, entramos em um território delicado. Se o dono da loja subir fotos cruas de 10 MB tiradas do celular, o servidor da Amazon (EC2) vai lotar rapidamente, e o site ficará incrivelmente lento para os clientes.

Nesta Parte 3, vamos aprender a dominar os arquivos de mídia no Django: como aceitar uploads, como guardar isso de forma isolada, e o mais avançado — **como forçar o Python a redimensionar e espremer a imagem automaticamente** toda vez que alguém tentar salvar um produto muito pesado!

---

## Passo 1: Preparando o Terreno (Pillow e Configurações)

O Django se recusa a trabalhar com campos de imagem se ele não tiver o seu fiel escudeiro instalado: a biblioteca **Pillow** (o motor processador visual do Python).

1. No terminal, dentro da sua bolha `(venv)`, instale-o:

```bash
pip install Pillow
```

2. Abra o cérebro central do seu projeto (`nano meu_projeto/settings.py`). Desça até o final do arquivo (perto de onde fica o `STATIC_URL`) e adicione as regras do "Cofre de Mídias" do usuário:

```python
# Essas linhas geralmente ficam no fim do settings.py
STATIC_URL = 'static/'

# --- ADICIONE ISTO AQUI PARA UPLOADS ---
import os # (Suba até o topo do arquivo e coloque 'import os' lá na linha 1 se não tiver!)

# URL pública que o cliente vai usar para acessar a foto
MEDIA_URL = '/media/'

# Pasta física real dentro do Amazon Linux onde as fotos serão salvas de verdade
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

---

## Passo 2: A Rota Pública das Mídias (`urls.py`)

No modo de desenvolvimento, o Django é meio preguiçoso e não entrega arquivos de mídia para a rua automaticamente se você não mandar ele fazer isso nas rotas principais.

Abra o arquivo principal de URLs (`nano meu_projeto/urls.py`) e deixe-o assim:

```python
from django.contrib import admin
from django.urls import path
from core import views

# --- NOVOS IMPORTS PARA LER IMAGENS ---
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.pagina_inicial, name='home'),
]

# --- ADICIONE ESTA MÁGICA NO FINAL ---
# Isso avisa para o Carteiro do Django: "Se alguém pedir um arquivo na URL /media/, vá buscar na pasta física MEDIA_ROOT"
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## Passo 3: O "Jeito Purista" (Criando o Campo e Otimizando a Imagem)

É aqui que a engenharia pesada acontece! Vamos modificar o nosso `Produto` lá no arquivo `core/models.py`.

Nós não vamos apenas adicionar o local da foto; vamos **hackear o botão de Salvar (save)** do Django. Antes dele encostar a foto no banco de dados, o nosso código vai interceptar o arquivo, abri-lo, cortá-lo e reduzi-lo para um tamanho aceitável!

Abra o arquivo `nano core/models.py` e substitua todo o modelo por este código cirúrgico:

```python
from django.db import models
from django.core.validators import FileExtensionValidator
from PIL import Image  # O motor do Pillow que instalamos!

class Produto(models.Model):
    nome = models.CharField(max_length=100)
    preco = models.DecimalField(max_digits=8, decimal_places=2)
    em_estoque = models.BooleanField(default=True)

    # NOVO: O campo de Imagem (Permitindo apenas PNG ou JPG)
    # Repare no blank=True, null=True (Significa que é opcional ter foto)
    imagem = models.ImageField(
        upload_to='produtos_fotos/',
        validators=[FileExtensionValidator(['jpg', 'jpeg', 'png'])],
        blank=True,
        null=True
    )

    def __str__(self):
        return self.nome

    # ==========================================
    # O SEGREDO PROFISSIONAL: INTERCEPTANDO O SAVE
    # ==========================================
    def save(self, *args, **kwargs):
        # 1. Deixa o Django salvar o produto original normalmente no banco primeiro
        super().save(*args, **kwargs)

        # 2. Se o produto tem uma imagem anexada a ele...
        if self.imagem:
            # Abre o arquivo de imagem que acabou de ser salvo na rocha
            img = Image.open(self.imagem.path)

            # 3. Verifica se a imagem é estupidamente gigante (Ex: Maior que 800 pixels de largura ou altura)
            if img.height > 800 or img.width > 800:
                tamanho_ideal = (800, 800)

                # O Pillow faz a matemática e espreme a foto sem distorcer a proporção!
                img.thumbnail(tamanho_ideal)

                # Salva a imagem por cima da antiga, forçando uma qualidade de 70% para não pesar no servidor
                img.save(self.imagem.path, quality=70, optimize=True)
```

_(Salve e feche o arquivo com `Ctrl+O`, `Enter`, `Ctrl+X`)._

### O impacto disso:

Se o administrador upar uma foto `.jpg` crua de 4.000 pixels pesando **5.5 MB**, o Django interceptará silenciosamente o processo. Em milissegundos, a imagem será reescrita no servidor Amazon Linux caindo para menos de **100 KB** (800x800). O seu site voará rápido como um foguete, e os usuários de celular agradecerão!

---

## Passo 4: Avisando o Banco de Acordo

Lembra da regra de ouro? Sempre que mexer no `models.py` e adicionar uma coluna nova (a imagem), o banco precisa ficar sabendo! No terminal:

```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

---

## Passo 5: Exibindo a Mídia Dinâmica na Vitrine HTML

Por último, como você faz aquela foto otimizada e levinha aparecer finalmente pro cliente lá na página inicial?

É surpreendentemente simples no sistema de Templates. Lembra do seu arquivo `core/templates/core/index.html`?

Onde ficaria a caixinha com as informações do produto, você simplesmente chamará a variável com `.url` no final, dessa forma:

```html
<!-- Dentro do seu laço de repetição {% for produto in lista_de_produtos %} -->

<div class="produto-card">
  <!-- CHAMANDO A IMAGEM AQUI (Com proteção caso o produto tenha sido cadastrado sem foto) -->
  {% if produto.imagem %}
  <img
    src="{{ produto.imagem.url }}"
    alt="{{ produto.nome }}"
    style="max-width: 100%; border-radius: 8px;"
  />
  {% else %}
  <!-- Caso o dono da loja tenha esquecido de subir uma foto, mostramos um aviso visual -->
  <p style="color: gray; font-style: italic;">Sem Imagem</p>
  {% endif %}

  <h3 style="margin: 0; color: #2c3e50;">{{ produto.nome }}</h3>
  <div class="preco">R$ {{ produto.preco }}</div>
</div>
```

**E pronto!**
Da próxima vez que você abrir o seu `/admin/`, clicar na tabela de Produtos e ir na edição, verá que o painel gerou automaticamente um lindo botão de **"Escolher Arquivo..."** no navegador para você upar suas fotos! Tudo 100% automático, otimizado nos bastidores do seu Amazon EC2 e instantaneamente visível com o Padrão MTV de programação.
