# ADR 0002 – Fuso `America/Fortaleza`, idioma `pt-br` e "ano atual" local

- **Status:** Aceito
- **Data:** 2026-11-27
- **Tarefas:** T03, T12 · **Requisitos:** RNF08 (novo), RF05

## Contexto

O projeto saiu do `startproject` com `LANGUAGE_CODE = "en-us"` e `TIME_ZONE = "UTC"`. Nenhum requisito falava de fuso ou idioma, mas duas regras dependem deles:

- A T12 limita `ano_lancamento` ao "ano atual" e pedia `timezone.now().year`. Com `USE_TZ = True`, `timezone.now()` devolve a hora em **UTC**. Em Fortaleza (UTC−3), das 21h às 0h de 31/12 o ano em UTC já é o seguinte: um álbum do ano que vem seria aceito por 3 horas.
- As mensagens padrão do Django/DRF (ex.: campo obrigatório) saíam em inglês, enquanto as mensagens das regras de negócio são em português.

## Decisão

- Criar o **RNF08**: `TIME_ZONE = "America/Fortaleza"`, `LANGUAGE_CODE = "pt-br"`, mantendo `USE_TZ = True` (datas continuam gravadas em UTC).
- O "ano atual" é `timezone.localdate().year`, calculado a cada validação, inclusive nos testes.
- Datas e horas gravadas (`avaliado_em`) continuam vindo de `timezone.now()`, para o `mock.patch` dos testes funcionar.

## Alternativas consideradas

- **Manter UTC e `timezone.now().year`**: mais simples, mas erra na virada do ano local.
- **`datetime.date.today()`**: usa o fuso do sistema operacional, não o do Django; o resultado mudaria de máquina para máquina.
- **Manter `en-us`**: as mensagens ficariam misturadas (inglês do framework, português das regras).

## Consequências

- Testes que conferem mensagens padrão esperam o texto em português (ex.: `"Este campo é obrigatório."`).
- Testes do ano não podem fixar o valor; calculam `timezone.localdate().year`.
- A T12, a "Decisões tomadas" do BACKLOG e o CLAUDE.md já foram ajustados no commit `573733f`.
