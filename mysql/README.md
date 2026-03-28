# Shared MySQL

A single MySQL 8.4 server shared across all local Laravel projects.

## Start

```bash
cd ~/docker/mysql
docker compose up -d
```

## Connection

| Setting  | Value      |
|----------|------------|
| Host     | `127.0.0.1` |
| Port     | `3306`     |
| Username | `root`     |
| Password | `password` |

## Adding a New Project

Create a database for the project:

```bash
docker exec mysql mysql -uroot -ppassword -e "CREATE DATABASE IF NOT EXISTS myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

The project's `compose.yaml` must join the `mysql-shared` network:

```yaml
services:
    laravel.test:
        networks:
            - mysql-shared

networks:
    mysql-shared:
        external: true
```

And the project's `.env` must point to this server:

```
DB_HOST=mysql
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=password
```

## Removing a Project's Database

```bash
docker exec mysql mysql -uroot -ppassword -e "DROP DATABASE myapp;"
```

## Data

MySQL data is stored in the `mysql_mysql-data` Docker volume. It persists across container restarts and is not tied to any individual project.
