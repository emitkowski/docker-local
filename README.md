# Local Docker Infrastructure

Shared Docker services used across all local Laravel projects.

## Services

| Service | Directory | Purpose |
|---------|-----------|---------|
| nginx-proxy | `proxy/` | SSL termination and routing for all local `.test` domains |
| MySQL | `mysql/` | Shared MySQL 8.4 server for all projects |
| Redis | `redis/` | Shared Redis cache server for all projects |
| Mailpit | `mailpit/` | Shared mail capture for all projects — dashboard at `http://localhost:8025` |

## Setup (new machine)

Start all services once from the `~/docker/` directory — they restart automatically after that.

```bash
cd ~/docker && docker compose up -d
```

> **Important:** always use `docker compose down && docker compose up -d` (not just `up -d`) when restarting services. If a container is recreated without `down` first — e.g. after resolving a port conflict — it may not rejoin its Docker network, making it unreachable from project containers and causing slow or broken app behaviour.

Then follow `NEW_PROJECT.md` in any project repo to wire up a new app.

## SSL Certificates

Certs are not stored in this repo — generate them per machine:

```bash
# Install mkcert CA (WSL + Windows browsers)
mkcert -install
powershell.exe -Command "Start-Process powershell -Verb RunAs -ArgumentList 'cd $(wslpath -w $(mkcert -CAROOT)); certutil -addstore -f Root rootCA.pem'"
```

Per-project certs are generated during project setup (see `NEW_PROJECT.md`).

## Database

See `mysql/README.md` for connection details and per-project database management.

## Troubleshooting

**Apps are slow or failing to connect to Redis/MySQL/Mailpit**

Check that each shared container is actually attached to its network:

```bash
docker network inspect redis-shared --format '{{range .Containers}}{{.Name}} {{end}}'
docker network inspect mailpit-shared --format '{{range .Containers}}{{.Name}} {{end}}'
docker network inspect mysql-shared --format '{{range .Containers}}{{.Name}} {{end}}'
```

Each line should include both the shared service container (`redis`, `mailpit`, `mysql`) and the project app containers. If a service is missing, recreate it:

```bash
cd ~/docker && docker compose down && docker compose up -d
```
