# CLAUDE.md

API REST pessoal para catalogar álbuns de música, dar notas e organizá-los por artista.
É um projeto de **aprendizado de Django REST Framework**, com um único usuário.

- O que o sistema deve fazer: [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) (RF01–RF15, RNF01–RNF08).
- Como e em que ordem construir: [docs/BACKLOG.md](docs/BACKLOG.md) (tarefas T01–T24, com critérios de aceite).
- Decisões de arquitetura: [docs/adr/](docs/adr/). Mudanças: [CHANGELOG.md](CHANGELOG.md) (atualize ao fechar cada tarefa).

Esses dois arquivos são a fonte da verdade. Se algo aqui conflitar com eles, valem eles. Se uma tarefa pedir algo que não está nos documentos, pergunte antes de inventar.

## Como trabalhar neste projeto

- **Uma tarefa do backlog por vez**, na ordem sugerida (E0 → E6). Antes de começar, leia os critérios de aceite (CA) da tarefa e as "Decisões tomadas" do backlog.
- **Não adiante tarefas futuras.** Ex.: em T06 não implemente a validação de T07 nem o 409 de T16.
- Ao terminar, marque os CA cumpridos com `[x]` no `BACKLOG.md`.
- Uma branch por tarefa (ex.: `t03-drf`), commit com prefixo da tarefa (ex.: `T03: configura o DRF`). Não faça commit nem push sem pedido.
- Como o objetivo é aprender, explique de forma curta o *porquê* das escolhas de DRF (serializer vs. view, validação, filtros), sem esconder a lógica em abstrações desnecessárias.
- Fique no escopo. **Fora do escopo:** recomendações, Last.fm/Spotify, faixas e playlists, múltiplos usuários, frontend/templates, deploy, histórico de notas.

## Definição de pronto (toda tarefa)

1. Todos os CA da tarefa cobertos por testes ou conferidos com um request.
2. `python manage.py test` passa.
3. Todo endpoint novo exige token (sem token → 401).
4. Erros de validação → 400 indicando o campo inválido.

## Stack e comandos

- Python + Django 6.1 + DRF + django-filter + python-dotenv. Banco SQLite (`db.sqlite3`). Só roda localmente.
- Dependências fixadas em `requirements.txt`; adicione novas lá com versão exata.

```bash
source .venv/bin/activate          # ou o virtualenv local
pip install -r requirements.txt
cp .env.example .env               # e preencha DJANGO_SECRET_KEY
python manage.py makemigrations biblioteca
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
python manage.py test              # roda todos os testes
python manage.py test biblioteca.tests.NomeDaClasse   # um grupo só
```

## Estrutura

- `setup/` – projeto Django (`settings.py`, `urls.py`).
- `biblioteca/` – o único app: models, serializers, views, filtros e testes.
- Rotas da API sob `/api/` via router do DRF. Token em `POST /api/token/` (`obtain_auth_token`).

## Convenções

- **Português sem acento** em URLs, nomes de campos, parâmetros e nomes de models: `artistas`, `albuns`, `ano_lancamento`, `avaliado_em`, `sem_nota`. Mensagens de erro ao usuário em português com acento.
- Segredos só por variável de ambiente (`DJANGO_SECRET_KEY`, carregada do `.env`). Nunca escreva chaves no código; `.env` e `db.sqlite3` não vão para o git.
- Configuração global do DRF (`REST_FRAMEWORK` em `settings.py`): só `TokenAuthentication`, `IsAuthenticated`, `PageNumberPagination` com `PAGE_SIZE = 20`, `DjangoFilterBackend` e `OrderingFilter`. Não sobrescreva isso por view sem motivo.
- Localização: `LANGUAGE_CODE = "pt-br"`, `TIME_ZONE = "America/Fortaleza"`, `USE_TZ = True` (RNF08). As mensagens padrão do Django/DRF saem em português (ex.: `"Este campo é obrigatório."`); testes que conferem essas mensagens devem esperar o texto em português.
- `INSTALLED_APPS` deve conter `rest_framework`, `rest_framework.authtoken` (caminho completo), `django_filters` e `biblioteca.apps.BibliotecaConfig`.
- `.env` carregado por `python-dotenv`; `DJANGO_SECRET_KEY` ausente ou vazia → `ImproperlyConfigured` (ADR 0001).

## Modelo de dados

Campos e constraints: BACKLOG T05 e T09. `avaliado_em` é somente leitura na API. Não existe tabela de histórico de notas.

## Regras de negócio (todas precisam de teste)

- **Unicidade sem diferenciar caixa**: nome do artista e título+artista. O DRF não valida sozinho constraints com `Lower()`; escreva a validação no serializer (`validate_nome`, `validate`), excluindo a própria instância na edição.
  - Mensagens: `"Já existe um artista com esse nome."` e `"Este artista já tem um álbum com esse título."`
- **Nota**: de 0 a 10 em passos de 0.5, opcional (`null` = "quero ouvir"). Valores inválidos → `{"nota": ["A nota deve ir de 0 a 10 em passos de 0.5."]}`. Use `Decimal`, não `float`, para checar o passo.
- **Ano de lançamento**: entre 1900 e o ano atual, calculado na hora no fuso local com `timezone.localdate().year` (nunca fixo, nem nos testes). Não use `timezone.now().year`: ele devolve o ano em UTC, que vira 3h antes da meia-noite local.
- **`avaliado_em`**: recebe `timezone.now()` quando a nota é dada pela primeira vez ou muda de valor; não muda se a mesma nota for reenviada ou se o `PATCH` não tocar em `nota`; volta a `null` quando a nota é removida.
- **Proteção do artista**: apagar artista com álbuns → **409** com `{"detail": "Artista possui álbuns e não pode ser apagado."}` (tratar `ProtectedError` no `destroy`).

## Consulta de álbuns

Filtros e ordenação: BACKLOG T17–T22 (um `AlbumFilter` do django-filter; `?artista=` inexistente → 400).

## Testes

- Em `biblioteca/tests.py` (ou um pacote `biblioteca/tests/` se crescer), com `APITestCase` e autenticação por token.
- Para `avaliado_em`, fixe a hora com `unittest.mock.patch("django.utils.timezone.now")`.
  Para o patch funcionar, o código precisa chamar `timezone.now()` (com `from django.utils import timezone`), nunca `from django.utils.timezone import now`.
- Teste os casos de borda listados nos CA (ex.: notas `7.3`, `11`, `-1`, `10.5`, `"abc"`; anos 1899, 1900, ano atual e ano atual + 1).
