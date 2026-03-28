# Shared Mailpit

A single Mailpit instance that captures all outbound mail from all local Laravel projects.

## Start

```bash
cd ~/docker/mailpit
docker compose up -d
```

## Access

| | |
|---|---|
| Dashboard | `http://localhost:8025` |
| SMTP | `localhost:1025` |

All projects send mail to this single inbox — useful for seeing emails across all apps in one place.

## Adding a New Project

The project's `compose.yaml` must join the `mailpit-shared` network:

```yaml
services:
    laravel.test:
        networks:
            - mailpit-shared

networks:
    mailpit-shared:
        external: true
```

And the project's `.env` must point to this server:

```
MAIL_HOST=mailpit
MAIL_PORT=1025
```
