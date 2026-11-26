# Biblioteca pessoal de música (API)

API REST em Django + Django REST Framework para cadastrar álbuns, dar notas e organizá-los por artista.

> Estado atual: projeto Django criado (`setup`) e app `biblioteca` registrado. Modelos, endpoints e autenticação ainda não foram implementados.

## Requisitos

- Python 3.12+ (exigido pelo Django 6.x)

## Como rodar

```bash
# 1. Criar e ativar o ambiente virtual
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# 2. Instalar as dependências
pip install -r requirements.txt

# 3. Criar o .env com uma DJANGO_SECRET_KEY nova
cp .env.example .env
python -c "from django.core.management.utils import get_random_secret_key as g; print(g())"
# cole a chave gerada em DJANGO_SECRET_KEY no .env

# 4. Criar o banco (SQLite)
python manage.py migrate

# 5. (Opcional) Criar um usuário para o admin
python manage.py createsuperuser

# 6. Subir o servidor
python manage.py runserver
```

O admin fica em http://127.0.0.1:8000/admin/.

## Testes

```bash
python manage.py test
```

## Estrutura

| Caminho | Conteúdo |
|---|---|
| `setup/` | Configuração do projeto Django (settings, urls, wsgi/asgi) |
| `biblioteca/` | App com os modelos e a API |
| `docs/REQUIREMENTS.md` | Requisitos funcionais |
| `docs/BACKLOG.md` | Tarefas, decisões e critérios de aceite |
