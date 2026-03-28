# nginx-proxy

Shared reverse proxy that routes all local `.test` domains to their respective Sail containers over HTTPS.

## Start

```bash
cd ~/docker/proxy
docker compose up -d
```

## How it works

- Listens on ports 80 and 443
- Automatically detects any Docker container with a `VIRTUAL_HOST` environment variable
- Routes HTTPS traffic using certs from `proxy/certs/`
- Each project's Sail `compose.yaml` sets `VIRTUAL_HOST: '${APP_DOMAIN}'` and joins the `nginx-proxy` external network

## Adding a new project

1. Generate a cert for the project's domain:

```bash
cd /path/to/project/docker/certs
mkcert -cert-file cert.pem -key-file key.pem myapp.test
```

2. Register it with the proxy:

```bash
cp cert.pem ~/docker/proxy/certs/myapp.test.crt
cp key.pem ~/docker/proxy/certs/myapp.test.key
docker exec nginx-proxy nginx -s reload
```

## Certs

SSL certificates live in `proxy/certs/` and are excluded from git. Regenerate them with `mkcert` on each machine.
