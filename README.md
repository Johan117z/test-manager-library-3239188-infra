# test-manager-library-3239188-infra

Infrastructure for **Manager Library**: containers, network, volume and an **empty** PostgreSQL 16 engine.

> Exam: ADSO 3239188, Thursday 2026-11-05, 7:00 p.m. to 11:00 p.m. (Colombia). Stack: React (`app`) + Laravel (`api`).
> Spec: [`test-manager-library-3239188-docs`](https://github.com/code-sena/test-manager-library-3239188-docs) (`spec-python/`).

## One-command startup

Requirements: Docker with Docker Compose v2.

```bash
cp .env.example .env      # then edit the values, especially POSTGRES_PASSWORD
docker compose up -d
```

That is all. `docker compose ps` shows `manager-library-db` as `healthy` after a few seconds.

| Service | Image | Host port | What it is |
|---|---|---|---|
| `db` | `postgres:16-alpine` | `${POSTGRES_PORT}` (default `5432`) | Empty database `${POSTGRES_DB}`, owned by `${POSTGRES_USER}`, server in UTC |

The data lives in the named volume `manager-library_db-data`.

Connection string for the API (`api/.env`):

```text
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1      # "db" when the API runs inside this compose network
DB_PORT=5432
DB_DATABASE=library
DB_USERNAME=library
DB_PASSWORD=<same as POSTGRES_PASSWORD>
```

## The database starts empty (ADR-001)

This repository creates **only** the engine, the database, the user and the volume. It contains **no table DDL** and no init
scripts. The schema `library`, the six tables, indexes, constraints and the five seeded categories belong to the **Laravel migrations**
in the `api` repo. The initial admin is created by the API at startup from `ADMIN_USERNAME` / `ADMIN_PASSWORD`, never by SQL.

Check that it is empty:

```bash
docker compose exec db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "\dt *.*"' | grep -v -E "pg_catalog|information_schema"
grep -riE "create[[:space:]]+table" . --exclude-dir=.git   # must print nothing
```

After running the API migrations, the queries in `data-model.md` §10 return what that document declares.

## Common commands

| Command | Effect |
|---|---|
| `docker compose up -d` | Start |
| `docker compose logs -f db` | Follow database logs |
| `docker compose down` | Stop, keep data |
| `docker compose down -v` | Stop and **delete** the volume (back to an empty database) |

## Secrets

The real `.env` is in `.gitignore`. Only `.env.example` is versioned, with placeholder values.

## Roadmap

T-20 adds the `api` (Laravel) and `app` (React) services to this same compose file, so one `docker compose up` starts everything.
