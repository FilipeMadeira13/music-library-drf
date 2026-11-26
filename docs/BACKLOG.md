# Backlog: API de biblioteca pessoal de música

Baseado em [REQUIREMENTS.md](REQUIREMENTS.md). Cada tarefa leva cerca de 1h e tem critérios de aceite (CA) que podem ser conferidos com um teste automatizado ou com um request.

## Decisões tomadas

| Tema | Decisão |
|---|---|
| `avaliado_em` | `DateTimeField`. Só muda quando a nota é dada pela primeira vez ou quando o valor dela muda. |
| Unicidade | O nome do artista e o par título+artista ignoram maiúsculas/minúsculas. |
| Apagar artista com álbuns | Responde **409 Conflict**. |
| Token | Obtido com `POST /api/token/` (`obtain_auth_token`). |
| Paginação | `PageNumberPagination` com `page_size = 20`. |
| Ordem padrão | Artistas por `nome`, álbuns por `titulo`. |
| `?genero=` | Igual, ignorando maiúsculas (`iexact`). |
| Idioma | URLs, campos e parâmetros em português, sem acento. |
| Filtros | `django-filter`. |
| `?sem_nota=false` | Devolve só os álbuns com nota. |
| Campo de ordenação do ano | `ano_lancamento`. |
| `?artista=<id>` inexistente | Responde 400. |

## Definição de pronto (vale para todas as tarefas)

- [ ] Todos os critérios de aceite da tarefa estão cobertos por testes ou foram conferidos com um request.
- [ ] `python manage.py test` passa.
- [ ] Todo endpoint novo exige token (sem token → 401).
- [ ] Os erros de validação respondem 400 e indicam o campo inválido.

---

## E0 – Setup

### T01 – Criar o projeto Django e o app `biblioteca`
**Requisitos:** RNF01, RNF02, RNF03 · **Depende de:** nenhuma

Criar o virtualenv, o projeto Django, o app `biblioteca` e o `requirements.txt` com Django, djangorestframework e django-filter. O banco é o SQLite padrão.

- [x] `pip install -r requirements.txt` instala tudo sem erro.
- [x] `python manage.py check` termina sem erros.
- [x] `python manage.py runserver` sobe em `http://127.0.0.1:8000/`.
- [x] O banco configurado é SQLite (`db.sqlite3`).
- [x] O `.gitignore` inclui o virtualenv, `.env`, `db.sqlite3` e `__pycache__/`.

### T02 – Ler a `SECRET_KEY` do ambiente
**Requisitos:** RNF07 · **Depende de:** T01

- [ ] `settings.py` lê a chave de `os.environ["DJANGO_SECRET_KEY"]`.
- [ ] Sem a variável, o Django para com `ImproperlyConfigured` e uma mensagem que diz qual variável falta.
- [ ] Nenhuma chave aparece no código (`grep -r "django-insecure" .` não encontra nada fora do virtualenv).
- [ ] Há um `.env.example` com `DJANGO_SECRET_KEY=` e uma instrução de como gerar a chave.

### T03 – Configurar o DRF
**Requisitos:** RNF04 · **Depende de:** T01

Em `INSTALLED_APPS`: `rest_framework`, `rest_framework.authtoken`, `django_filters` e `biblioteca`. Em `REST_FRAMEWORK`: `TokenAuthentication`, `IsAuthenticated`, `PageNumberPagination` com `PAGE_SIZE = 20`, `DjangoFilterBackend` e `OrderingFilter`.

- [ ] `python manage.py migrate` cria a tabela `authtoken_token`.
- [ ] `TokenAuthentication` é a única classe de autenticação padrão.
- [ ] `IsAuthenticated` é a permissão padrão.
- [ ] A paginação e os backends de filtro estão configurados como padrão.

### T04 – Endpoint para obter o token
**Requisitos:** RNF04 · **Depende de:** T03

`POST /api/token/` com `username` e `password`, usando `obtain_auth_token`.

- [ ] Com credenciais válidas → 200 com `{"token": "<chave>"}`.
- [ ] Duas chamadas com o mesmo usuário devolvem o mesmo token.
- [ ] Com senha errada → 400.
- [ ] Sem campos → 400, indicando `username` e `password`.
- [ ] Os testes cobrem esses casos.

---

## E1 – Artistas

### T05 – Model `Artista`
**Requisitos:** RF01, RF02 · **Depende de:** T01

Campo `nome` (`CharField`). `UniqueConstraint(Lower("nome"))`. `Meta.ordering = ["nome"]`.

- [ ] A migração é criada e aplicada sem erro.
- [ ] `str(artista)` devolve o nome.
- [ ] Teste: criar "Radiohead" e depois "radiohead" direto no ORM levanta `IntegrityError`.

### T06 – CRUD de `/api/artistas/`
**Requisitos:** RF01 · **Depende de:** T03, T05

`ArtistaSerializer` (`id`, `nome`) + `ModelViewSet` registrado no router em `/api/`.

- [ ] `POST /api/artistas/` → 201 com o artista criado.
- [ ] `GET /api/artistas/` → 200, paginado (`count`, `next`, `previous`, `results`) e ordenado por nome.
- [ ] `GET /api/artistas/<id>/` → 200.
- [ ] `PUT` e `PATCH /api/artistas/<id>/` → 200 com os dados atualizados.
- [ ] `DELETE /api/artistas/<id>/` de um artista sem álbuns → 204.
- [ ] `GET`, `PUT` ou `DELETE` de um id inexistente → 404.

### T07 – Validar o nome do artista
**Requisitos:** RF02, RNF05 · **Depende de:** T06

O DRF não valida sozinho uma constraint com `Lower()`, então é preciso escrever `validate_nome`.

- [ ] Sem `nome` → 400 com a chave `nome`.
- [ ] `nome` vazio ou só com espaços → 400 com a chave `nome`.
- [ ] Nome duplicado, com qualquer caixa → 400 com `{"nome": ["Já existe um artista com esse nome."]}`.
- [ ] Editar um artista mantendo o próprio nome → 200.
- [ ] Editar um artista para o nome de outro → 400.

### T08 – Exigir autenticação em `/api/artistas/`
**Requisitos:** RNF04 · **Depende de:** T04, T06

- [ ] Sem o header `Authorization` → 401.
- [ ] `Authorization: Token invalido` → 401.
- [ ] Com um token válido → 200.
- [ ] Os testes cobrem lista, detalhe, criação e remoção.

---

## E2 – Álbuns

### T09 – Model `Album`
**Requisitos:** RF04, RF05, RF06 · **Depende de:** T05

| Campo | Tipo |
|---|---|
| `titulo` | `CharField` |
| `artista` | `ForeignKey(Artista, on_delete=PROTECT, related_name="albuns")` |
| `ano_lancamento` | `PositiveSmallIntegerField(null=True, blank=True)` |
| `genero` | `CharField(blank=True)` |
| `nota` | `DecimalField(max_digits=3, decimal_places=1, null=True, blank=True)` |
| `avaliado_em` | `DateTimeField(null=True, blank=True)` |

`UniqueConstraint(Lower("titulo"), "artista")`. `Meta.ordering = ["titulo"]`.

- [ ] A migração é criada e aplicada sem erro.
- [ ] Teste: dois álbuns "OK Computer" e "ok computer" do mesmo artista no ORM → `IntegrityError`.
- [ ] Teste: o mesmo título com artistas diferentes é aceito.
- [ ] Teste: apagar no ORM um artista que tem álbuns levanta `ProtectedError`.

### T10 – CRUD de `/api/albuns/`
**Requisitos:** RF04, RNF04 · **Depende de:** T09

- [ ] `POST`, `GET` (lista e detalhe), `PUT`, `PATCH` e `DELETE` respondem 201, 200, 200, 200 e 204.
- [ ] Id inexistente → 404.
- [ ] `artista` é enviado e devolvido como id.
- [ ] A lista é paginada e ordenada por título.
- [ ] `avaliado_em` é somente leitura: o valor enviado pelo cliente é ignorado.
- [ ] Sem token → 401.

### T11 – Campos obrigatórios do álbum
**Requisitos:** RF05, RNF05 · **Depende de:** T10

- [ ] Sem `titulo` → 400 com a chave `titulo`.
- [ ] Sem `artista` → 400 com a chave `artista`.
- [ ] `artista` com id inexistente → 400 com a chave `artista`.
- [ ] Só com `titulo` e `artista` → 201, com `ano_lancamento`, `nota` e `avaliado_em` nulos e `genero` vazio.

### T12 – Validar o ano de lançamento
**Requisitos:** RF05, RNF05 · **Depende de:** T10

- [ ] `ano_lancamento = 1900` → aceito.
- [ ] `ano_lancamento` igual ao ano atual → aceito.
- [ ] `1899` → 400 com a chave `ano_lancamento`.
- [ ] O ano atual + 1 → 400 com a chave `ano_lancamento`.
- [ ] `null` ou campo ausente → aceito.
- [ ] O ano atual é calculado na hora (`timezone.now().year`), não fica fixo no código. Os testes também calculam o ano.

### T13 – Título único por artista
**Requisitos:** RF06, RNF05 · **Depende de:** T10

- [ ] Mesmo título, com qualquer caixa, para o mesmo artista → 400 com uma mensagem clara (ex.: "Este artista já tem um álbum com esse título.").
- [ ] Mesmo título para outro artista → 201.
- [ ] Editar um álbum sem mudar o título → 200.
- [ ] Mudar o artista de um álbum para um que já tem esse título → 400.

---

## E3 – Avaliação

### T14 – Validar a nota
**Requisitos:** RF07, RF08, RNF05 · **Depende de:** T10

- [ ] Aceita `0`, `0.5`, `7`, `7.5` e `10`.
- [ ] Aceita `null` e campo ausente ("quero ouvir").
- [ ] Recusa `7.3`, `11`, `-1`, `10.5` e `"abc"` → 400 com `{"nota": ["A nota deve ir de 0 a 10 em passos de 0.5."]}`, ou com uma mensagem igualmente clara para valores que não são números.
- [ ] A validação está no serializer ou em um validador reutilizável e é coberta por testes.

### T15 – Preencher `avaliado_em` automaticamente
**Requisitos:** RF09, RF10 · **Depende de:** T14

Os testes fixam a hora com `unittest.mock.patch` em `django.utils.timezone.now`.

- [ ] Criar com nota → `avaliado_em` recebe a hora atual.
- [ ] Criar sem nota → `avaliado_em` fica `null`.
- [ ] Dar nota a um álbum que não tinha → `avaliado_em` recebe a hora atual.
- [ ] Mudar a nota (7 → 8) → `avaliado_em` recebe a hora nova.
- [ ] Reenviar a mesma nota (7.5 → 7.5) → `avaliado_em` não muda.
- [ ] `PATCH` de outro campo, sem `nota` → `avaliado_em` não muda.
- [ ] Mudar a nota para `null` → `avaliado_em` volta a `null`.
- [ ] Só a nota atual é guardada; não existe model ou tabela de histórico (RF10).

---

## E4 – Proteção do artista

### T16 – Recusar a remoção de artista com álbuns
**Requisitos:** RF03 · **Depende de:** T06, T09

Tratar `ProtectedError` no `destroy` do `ArtistaViewSet`.

- [ ] `DELETE` de um artista com álbuns → 409 com `{"detail": "Artista possui álbuns e não pode ser apagado."}`.
- [ ] Depois do 409, o artista e os álbuns continuam no banco.
- [ ] Depois de apagar todos os álbuns dele, o `DELETE` do artista → 204.

---

## E5 – Consulta

### T17 – Filtrar por artista (`?artista=<id>`)
**Requisitos:** RF11 · **Depende de:** T03, T10

Criar um `AlbumFilter` (django-filter) ligado ao `AlbumViewSet`.

- [ ] `?artista=<id>` devolve só os álbuns desse artista.
- [ ] Artista sem álbuns → lista vazia (200).
- [ ] `?artista=` com id inexistente → 400.

### T18 – Lista "quero ouvir" (`?sem_nota=`)
**Requisitos:** RF12 · **Depende de:** T17

`BooleanFilter(field_name="nota", lookup_expr="isnull")`.

- [ ] `?sem_nota=true` → só os álbuns sem nota.
- [ ] `?sem_nota=false` → só os álbuns com nota.
- [ ] Sem o parâmetro → todos os álbuns.
- [ ] Valor inválido (ex.: `?sem_nota=talvez`) → 400.

### T19 – Filtrar por gênero (`?genero=`)
**Requisitos:** RF13 · **Depende de:** T17

- [ ] `?genero=rock` encontra os álbuns com gênero "Rock" e "ROCK".
- [ ] `?genero=rock` não encontra "Indie Rock".
- [ ] Gênero sem álbuns → lista vazia (200).

### T20 – Ordenar (`?ordering=`)
**Requisitos:** RF14 · **Depende de:** T10

`ordering_fields = ["nota", "ano_lancamento", "titulo"]`.

- [ ] `nota`, `-nota`, `ano_lancamento`, `-ano_lancamento`, `titulo` e `-titulo` ordenam corretamente.
- [ ] Em `?ordering=-nota`, os álbuns sem nota aparecem no fim.
- [ ] Um campo não permitido (ex.: `?ordering=genero`) é ignorado e vale a ordem padrão por título.

### T21 – Combinar filtros e ordenação
**Requisitos:** RF11 a RF14 · **Depende de:** T17, T18, T19, T20

- [ ] `?sem_nota=false&genero=rock&ordering=-nota` devolve só os álbuns de rock com nota, da maior para a menor.
- [ ] `?artista=<id>&sem_nota=true` devolve só os álbuns sem nota desse artista.
- [ ] Os filtros funcionam junto com a paginação (`count` reflete o filtro).

### T22 – Álbuns de um artista (`GET /api/artistas/<id>/albuns/`)
**Requisitos:** RF15 · **Depende de:** T06, T10

`@action(detail=True)` no `ArtistaViewSet`, usando o `AlbumSerializer`.

- [ ] Devolve só os álbuns do artista, paginados e ordenados por título.
- [ ] Artista sem álbuns → 200 com lista vazia.
- [ ] Artista inexistente → 404.
- [ ] Sem token → 401.

---

## E6 – Fechamento

### T23 – README
**Requisitos:** RNF03, RNF07 · **Depende de:** T04

- [ ] Explica como criar o virtualenv e instalar as dependências.
- [ ] Explica como definir `DJANGO_SECRET_KEY`.
- [ ] Explica `migrate`, `createsuperuser` e `runserver`.
- [ ] Mostra como obter o token e usá-lo, com exemplos `curl` para criar um artista, criar um álbum, dar nota e usar os filtros.

### T24 – Roteiro de aceite e cobertura dos testes
**Requisitos:** RNF06, "Como sei que funciona" · **Depende de:** todas

- [ ] Roteiro manual executado com sucesso:
  - Cadastrar um artista e alguns álbuns dele usando um token.
  - Dar nota a um álbum; as notas 7.3, 11 e -1 são recusadas.
  - Ver a lista "quero ouvir" (`?sem_nota=true`) e os álbuns por nota (`?ordering=-nota`).
  - Ver todos os álbuns de um artista (`/api/artistas/<id>/albuns/`).
- [ ] Existe pelo menos um teste para cada regra de negócio: nota (T14), unicidade (T07, T13), data de avaliação (T15) e proteção do artista (T16).
- [ ] `python manage.py test` passa.

---

## Ordem sugerida

```
E0: T01 → T02 → T03 → T04
E1: T05 → T06 → T07 → T08
E2: T09 → T10 → T11 → T12 → T13
E3: T14 → T15
E4: T16
E5: T17 → T18 → T19 → T20 → T21 → T22
E6: T23 → T24
```

## Matriz de rastreabilidade

| Requisito | Tarefas |
|---|---|
| RF01 | T05, T06 |
| RF02 | T05, T07 |
| RF03 | T09, T16 |
| RF04 | T09, T10 |
| RF05 | T09, T11, T12 |
| RF06 | T09, T13 |
| RF07 | T14 |
| RF08 | T14 |
| RF09 | T15 |
| RF10 | T15 |
| RF11 | T17, T21 |
| RF12 | T18, T21 |
| RF13 | T19, T21 |
| RF14 | T20, T21 |
| RF15 | T22 |
| RNF01 | T01 |
| RNF02 | T01 |
| RNF03 | T01, T23 |
| RNF04 | T03, T04, T08, T10, T22 |
| RNF05 | T07, T11, T12, T13, T14 |
| RNF06 | T24 (e os testes de cada tarefa) |
| RNF07 | T02, T23 |
