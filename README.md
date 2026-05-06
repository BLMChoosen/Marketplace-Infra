# Bloodmoon Marketplace Infra

Top-level Docker Compose orchestration for the Bloodmoon Marketplace platform.
Run this repo when you want the full stack: frontend, backend, database, cache,
broker, search, reverse proxy and observability.

## Expected layout

The three repositories should be siblings:

```text
Marketplace-Backend/
Marketplace-Frontend/
Marketplace-Infra/
```

`Marketplace-Infra/docker-compose.yml` includes the compose files from the
frontend and backend repos.

## Services

- `frontend`: Next.js standalone server
- `backend`: Flask API through Gunicorn/eventlet
- `celery_worker`: background task processor
- `db`: PostgreSQL 16
- `redis`: Redis 7
- `elasticsearch`: Elasticsearch 8
- `rabbitmq`: RabbitMQ 3 with management UI
- `nginx`: public reverse proxy for frontend, API and Socket.IO
- `prometheus`: metrics collection
- `grafana`: dashboards

MongoDB is not started by Docker Compose. Chat storage uses MongoDB Atlas via
`MONGODB_ATLAS_URI` in `Marketplace-Backend/.env`.

## Environment

Prepare environment files in each repo:

```powershell
Copy-Item ..\Marketplace-Backend\.env.example ..\Marketplace-Backend\.env
Copy-Item ..\Marketplace-Frontend\.env.local.example ..\Marketplace-Frontend\.env.local
Copy-Item .env.example .env
```

Fill production values before starting the stack. The backend requires real
secret keys, Atlas and Stripe values in production.

## Run

```powershell
docker compose config --quiet
docker compose up --build
```

After the containers are healthy:

```powershell
docker compose exec backend flask db upgrade
```

Public entry points:

- Marketplace: `http://localhost`
- API health: `http://localhost/api/health`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3001`
- RabbitMQ management: `http://localhost:15672`

## Production notes

- Put TLS termination in front of Nginx or extend `nginx/nginx.conf` with HTTPS.
- Replace example passwords in `.env` before exposing the stack.
- Keep `FRONTEND_ORIGIN` and `CORS_ORIGINS` aligned with the public domain.
- Leave `NEXT_PUBLIC_API_URL` and `NEXT_PUBLIC_WS_URL` empty when using Nginx
  same-origin routing.
- Rebuild the frontend image after changing any `NEXT_PUBLIC_*` variable.
- Keep MongoDB Atlas outside Compose; no local Mongo service is expected.

## Validation

Useful checks:

```powershell
docker compose config --quiet
docker compose config | Select-String -Pattern 'MONGO_URL|mongo_data|mongo:27017'
```

The second command should return no matches.
