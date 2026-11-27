# Progresso

Diário curto das sessões: o que foi feito, onde paramos e o que vem depois.
Atualize ao encerrar cada sessão (a entrada mais recente fica no topo).

## 2026-11-27

### Feito
- **T03 – Configurar o DRF** concluída e mergeada em `main` (PR #2, merge `d139af6`).
  - `573733f` T03: configura o DRF — `INSTALLED_APPS`, `REST_FRAMEWORK` global, `pt-br` e `America/Fortaleza` (RNF08), chave vazia → `ImproperlyConfigured`; ajustes em REQUIREMENTS e BACKLOG.
  - `23ee712` T03: adiciona CLAUDE.md, CHANGELOG e ADR 0001.
- ADR 0002 escrita para registrar a escolha de fuso/idioma e do `timezone.localdate().year` (já aplicada em `573733f`).
- Status das tarefas adicionado ao BACKLOG.

### Anteriores (2026-11-26)
- **T01** – `d768eac` first commit; `128e7df` move `SECRET_KEY` para `.env` e adiciona README.
- **T02** – `864ae6a` lê `DJANGO_SECRET_KEY` do ambiente (PR #1, merge `7e9b12f`).

### Onde paramos
- `main` limpo e sincronizado com `origin/main`. E0 com T01–T03 prontas; falta a T04.
- `setup/urls.py` só tem `admin/`; ainda não há rotas `/api/`.
- `biblioteca/` ainda sem models; `python manage.py test` roda **0 testes**.

### Próximos passos
1. **T04 – Endpoint do token**: branch `t04-token`; `POST /api/token/` com `obtain_auth_token` em `setup/urls.py`; testes em `biblioteca/tests.py` para os 4 CA (200 com token, mesmo token em duas chamadas, senha errada → 400, sem campos → 400 com `username` e `password`, mensagens em português).
2. Depois, E1: T05 (model `Artista`) → T06 → T07 → T08.

### Dúvidas em aberto
- **Pasta das ADRs:** o pedido de hoje citou `docs/decisoes/`, mas o projeto usa `docs/adr/`. Mantida `docs/adr/`; confirmar se é para renomear.
- **Testes da T03:** os CA foram conferidos por comando (`migrate`, `check`), sem teste automatizado. Vale um teste simples de configuração (ex.: `REST_FRAMEWORK` e `LANGUAGE_CODE`) ou basta a conferência manual?
- **Branches locais** `t02-secret-key` e `t03-drf` já mergeadas: apagar ou manter?
