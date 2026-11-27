# ADR 0001 – Carregar o `.env` com python-dotenv e recusar chave vazia

- **Status:** Aceito
- **Data:** 2026-11-27
- **Tarefas:** T01, T02 · **Requisitos:** RNF07

## Contexto

O RNF07 exige que a `SECRET_KEY` venha de variável de ambiente. A T01 listava só Django, djangorestframework e django-filter como dependências, e a T02 pedia ler `os.environ["DJANGO_SECRET_KEY"]`, mas nenhum documento dizia **como** a variável chega ao processo. O Django não lê arquivos `.env` sozinho.

Na prática, duas decisões foram tomadas no código sem registro:

1. O commit `128e7df` adicionou `python-dotenv` e o `settings.py` passou a chamar `load_dotenv(BASE_DIR / '.env')`.
2. Na T03, `os.environ["DJANGO_SECRET_KEY"]` virou `os.environ.get(...)` com `if not SECRET_KEY`. Uma variável definida mas vazia (o `.env.example` traz `DJANGO_SECRET_KEY=`) também passa a ser erro.

## Decisão

- Usar `python-dotenv` (versão fixa no `requirements.txt`) para carregar o `.env` da raiz do projeto ao importar o `settings.py`.
- `load_dotenv` roda com o padrão `override=False`: uma variável já exportada no shell vale mais que o `.env`.
- Uma `DJANGO_SECRET_KEY` ausente **ou vazia** levanta `ImproperlyConfigured("Defina a variável de ambiente DJANGO_SECRET_KEY.")`.

## Alternativas consideradas

- **Exportar a variável no shell** (`export DJANGO_SECRET_KEY=...`): não precisa de dependência, mas é preciso repetir a cada terminal e é fácil esquecer.
- **django-environ**: faz mais coisas (tipos, `DATABASE_URL`), mas é mais do que um projeto local com uma variável precisa.
- **Aceitar chave vazia**: o Django já recusa `SECRET_KEY` vazia em alguns pontos, mas com uma mensagem menos clara. Copiar o `.env.example` sem preencher daria esse erro confuso.

## Consequências

- `cp .env.example .env` e preencher a chave basta para rodar; o README já explica isso.
- Uma dependência a mais (`python-dotenv`), que precisa ser citada na T01 e no CLAUDE.md (já está no CLAUDE.md).
- O CA da T02 "lê de `os.environ["DJANGO_SECRET_KEY"]`" precisa ser reescrito para refletir o `.get` e a recusa da chave vazia.
- Nos testes a chave vem do `.env` local; não há valor padrão escondido no código.
