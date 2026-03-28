# Shared Redis

A single Redis server shared across all local Laravel projects.

## Start

```bash
cd ~/docker/redis
docker compose up -d
```

## Connection

| Setting | Value       |
|---------|-------------|
| Host    | `127.0.0.1` |
| Port    | `6379`      |

## Adding a New Project

The project's `compose.yaml` must join the `redis-shared` network:

```yaml
services:
    laravel.test:
        networks:
            - redis-shared

networks:
    redis-shared:
        external: true
```

And the project's `.env` must point to this server:

```
REDIS_HOST=redis
REDIS_PORT=6379
```

## Data

Redis data is stored in the `redis_redis-data` Docker volume and persists across restarts.
