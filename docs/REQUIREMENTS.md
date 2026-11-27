# Requisitos: API de biblioteca pessoal de música

## Objetivo
Uma API REST para eu cadastrar os álbuns que ouvi ou quero ouvir, dar notas e organizá-los por artista.
Objetivo de aprendizado: Django REST Framework.

## Usuários
Só eu (usuário único).

## Requisitos funcionais

### Artistas
- RF01: Cadastrar, listar, ver, editar e apagar artistas.
- RF02: O nome do artista é obrigatório e único.
- RF03: Não é possível apagar um artista que tem álbuns (a API responde com erro).

### Álbuns
- RF04: Cadastrar, listar, ver, editar e apagar álbuns.
- RF05: Campos do álbum:

| Campo | Obrigatório | Regra |
|---|---|---|
| título | sim | texto |
| artista | sim | um artista existente |
| ano de lançamento | não | inteiro entre 1900 e o ano atual |
| gênero | não | texto livre |
| nota | não | ver RF07 |
| avaliado_em | — | preenchido pela API |

- RF06: Não pode haver dois álbuns com o mesmo título para o mesmo artista.

### Avaliação
- RF07: A nota vai de 0 a 10, em passos de 0.5 (ex.: 7.5 é válido, 7.3 não).
- RF08: A nota é opcional. Álbum sem nota significa "quero ouvir".
- RF09: Ao dar ou mudar a nota, a API grava a data em `avaliado_em`. Ao remover a nota, `avaliado_em` volta a ficar vazio.
- RF10: Só a nota atual é guardada (sem histórico).

### Organização e consulta
- RF11: Filtrar álbuns por artista (`?artista=<id>`).
- RF12: Filtrar álbuns sem nota, ou seja, a lista "quero ouvir" (`?sem_nota=true`).
- RF13: Filtrar álbuns por gênero (`?genero=`).
- RF14: Ordenar álbuns por nota, ano ou título (`?ordering=-nota`).
- RF15: Listar os álbuns de um artista (`GET /api/artistas/<id>/albuns/`).

## Requisitos não funcionais
- RNF01: Python + Django + Django REST Framework.
- RNF02: Banco SQLite.
- RNF03: Roda só localmente (`python manage.py runserver`).
- RNF04: Todos os endpoints exigem autenticação por token (`TokenAuthentication`). Request sem token recebe 401.
- RNF05: Erros de validação respondem 400 com uma mensagem clara do campo inválido.
- RNF06: As regras de negócio (nota, unicidade, data de avaliação, proteção do artista) têm testes automatizados (`python manage.py test`).
- RNF07: Segredos (`SECRET_KEY`) lidos de variável de ambiente, nunca no código.
- RNF08: Fuso horário `America/Fortaleza` e idioma `pt-br` (mensagens padrão do Django/DRF em português). O "ano atual" é o ano no fuso local.

## Fora do escopo
- Recomendações e integração com APIs externas (Last.fm, Spotify).
- Faixas/músicas individuais e playlists.
- Vários usuários, rede social.
- Frontend ou templates HTML.
- Deploy online.
- Histórico de notas.

## Como sei que funciona
- Consigo cadastrar um artista e alguns álbuns dele pela API, usando um token.
- Consigo dar nota a um álbum, e notas inválidas (7.3, 11, -1) são recusadas.
- Consigo ver minha lista "quero ouvir" e meus álbuns ordenados pela nota.
- Consigo ver todos os álbuns de um artista.
- `python manage.py test` passa.
