# Task Manager API

API REST para gerenciamento de tarefas, construída como projeto de estudo de **Laravel**, **PostgreSQL** e **Docker**. Não há frontend: a interação é feita via HTTP (Postman, Insomnia, curl etc.).

## Stack

| Camada         | Tecnologia                  |
| -------------- | --------------------------- |
| Linguagem      | PHP 8.4                     |
| Framework      | Laravel 13                  |
| Banco de dados | PostgreSQL 17               |
| Servidor web   | Nginx 1.27 + PHP-FPM        |
| Autenticação   | Laravel Sanctum (planejado) |
| Ambiente       | Docker Compose              |

## Arquitetura

Fluxo de uma requisição:

```
Route → Middleware (Sanctum) → FormRequest → Controller → Policy → Service → Model → Resource
```

- **FormRequest**: valida os dados de entrada.
- **Controller**: fino, só orquestra a requisição.
- **Policy**: garante que o usuário só acessa as próprias tarefas.
- **Service**: concentra a regra de negócio.
- **Resource**: padroniza o JSON de resposta.

Não há camada de Repository na v1: o Eloquent já cumpre esse papel.

## Modelo de dados

```
users 1 ──── N tasks
```

### Tabela `tasks`

| Coluna        | Tipo (PostgreSQL)              | Obrigatório | Observação                                       |
| ------------- | ------------------------------ | ----------- | ------------------------------------------------ |
| `id`          | `bigserial` (PK)               | sim         | gerado automaticamente                           |
| `user_id`     | `bigint` (FK → `users.id`)     | sim         | dono da tarefa; apagado em cascata com o usuário |
| `title`       | `varchar(255)`                 | sim         |                                                  |
| `description` | `text`                         | não         |                                                  |
| `status`      | `varchar(20)`                  | sim         | `pending` (padrão), `in_progress`, `done`        |
| `due_date`    | `date`                         | não         | prazo da tarefa                                  |
| `created_at`  | `timestamp`                    | —           | preenchido pelo Laravel                          |
| `updated_at`  | `timestamp`                    | —           | preenchido pelo Laravel                          |

Índice composto em `(user_id, status)` para a listagem "minhas tarefas por status".

## Como rodar

Pré-requisito: Docker Engine com o plugin Compose (no WSL2).

```bash
# 1. Variáveis de ambiente
cp .env.example .env

# 2. Subir os containers (app, nginx, db)
docker compose up -d --build

# 3. Dependências e chave da aplicação
docker compose exec app composer install
docker compose exec app php artisan key:generate

# 4. Criar as tabelas
docker compose exec app php artisan migrate
```

A API fica em `http://localhost:8080`.

O PostgreSQL fica exposto em `localhost:5433` (usuário, senha e banco definidos no `.env`), para acesso por um cliente como o DBeaver ou pelo `psql`:

```bash
docker compose exec db psql -U task_manager -d task_manager
```

## Comandos úteis

```bash
docker compose exec app php artisan migrate --pretend   # mostra o SQL sem executar
docker compose exec app php artisan migrate:status      # quais migrations já rodaram
docker compose exec app php artisan migrate:rollback    # desfaz o último lote
docker compose exec app php artisan test                # roda os testes
```

## Progresso

- [x] Projeto Laravel 13 + Dockerfile PHP 8.4
- [x] Docker Compose com Nginx e PostgreSQL
- [x] Migration da tabela `tasks`
- [ ] Autenticação com Sanctum
- [ ] Model `Task` e relacionamento com `User`
- [ ] CRUD de tarefas (FormRequest, Controller, Policy, Service, Resource)
- [ ] Testes de feature
