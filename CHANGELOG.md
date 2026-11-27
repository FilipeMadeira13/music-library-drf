# Changelog

Mudanças relevantes do projeto, da mais recente para a mais antiga.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/). Como o projeto não tem versões, cada bloco corresponde a uma tarefa do [backlog](docs/BACKLOG.md).

## [Não lançado] – T03 (branch `t03-drf`)

### Adicionado
- DRF configurado: `rest_framework`, `rest_framework.authtoken` e `django_filters` em `INSTALLED_APPS`.
- `REST_FRAMEWORK` global com `TokenAuthentication`, `IsAuthenticated`, `PageNumberPagination` (`PAGE_SIZE = 20`), `DjangoFilterBackend` e `OrderingFilter`.
- Tabela `authtoken_token` criada pelo `migrate`.
- RNF08 nos requisitos: fuso `America/Fortaleza` e idioma `pt-br`; o "ano atual" é o ano local.
- `CLAUDE.md` com as instruções do projeto para o Claude Code.
- `docs/adr/0001-python-dotenv-e-chave-vazia.md` (aceito).

### Alterado
- `LANGUAGE_CODE` de `en-us` para `pt-br`: as mensagens padrão do Django/DRF passam a sair em português.
- `TIME_ZONE` de `UTC` para `America/Fortaleza` (`USE_TZ = True` mantido; datas continuam gravadas em UTC).
- T12: o ano atual passa a ser `timezone.localdate().year` (antes `timezone.now().year`, que usa o ano em UTC).
- `DJANGO_SECRET_KEY` vazia agora também levanta `ImproperlyConfigured` (antes só a variável ausente).
- `.env.example` com `DJANGO_SECRET_KEY=` vazio, como pede o CA da T02.
- Docs: RNF08 na matriz de rastreabilidade e na T12; T10 passa a depender da T03; CA da T01/T02 citam python-dotenv e a chave vazia; RF15 com o prefixo `/api/`.
- CLAUDE.md enxugado: model e filtros apontam para o BACKLOG (T05, T09, T17–T22); aviso sobre como chamar `timezone.now()` para o patch dos testes funcionar.

## T02 – Ler a `SECRET_KEY` do ambiente – 2026-11-26

### Adicionado
- `python-dotenv` no `requirements.txt`; o `settings.py` carrega o `.env` da raiz.
- `.env.example` com a instrução para gerar a chave.
- `README.md` com instalação, execução e testes.

### Alterado
- `SECRET_KEY` lida de `DJANGO_SECRET_KEY`; sem ela o Django para com `ImproperlyConfigured`.

### Removido
- Chave `django-insecure-...` do código.

## T01 – Criar o projeto Django e o app `biblioteca` – 2026-11-26

### Adicionado
- Projeto Django `setup` e app `biblioteca` registrado.
- `requirements.txt` com Django, djangorestframework e django-filter em versões fixas.
- Banco SQLite (`db.sqlite3`) e `.gitignore` com virtualenv, `.env`, `db.sqlite3` e `__pycache__/`.
- `docs/REQUIREMENTS.md` e `docs/BACKLOG.md`.
