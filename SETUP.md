
# Local Development Setup

Full instructions for recreating the local development environment on a new machine. Follow in order.

## 1. Prerequisites

### WSL2
Ensure WSL2 is installed with Ubuntu. All commands below run in WSL unless noted.

### Docker Desktop
Install Docker Desktop for Windows with WSL2 backend enabled.

### Git
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### mkcert
```bash
sudo apt install mkcert

# Trust the CA in both WSL and Windows browsers
mkcert -install
powershell.exe -Command "Start-Process powershell -Verb RunAs -ArgumentList 'cd $(wslpath -w $(mkcert -CAROOT)); certutil -addstore -f Root rootCA.pem'"
```

---

## 2. Shared Docker Infrastructure

Clone this repo and start all shared services:

```bash
git clone git@github.com:YOUR_ORG/dev-infrastructure.git ~/docker
cd ~/docker && docker compose up -d
```

Verify all are running:

```bash
docker ps
```

You should see `nginx-proxy` and `mysql` containers running.

---

## 3. Projects

Clone each project and follow its `NEW_PROJECT.md` for full setup. Key steps are summarised below for each.

### replica
The Laravel starter template — set up first as it's the reference.

```bash
git clone git@github.com:YOUR_ORG/replica.git ~/code/replica
cd ~/code/replica
```

Follow `NEW_PROJECT.md` in full.

---

### advisor
```bash
git clone git@github.com:YOUR_ORG/advisor.git ~/code/advisor
cd ~/code/advisor
cp .env.example .env
```

Update `.env`:
| Variable | Value |
|---|---|
| `APP_NAME` | `Advisor` |
| `APP_URL` | `https://advisor.test` |
| `APP_DOMAIN` | `advisor.test` |
| `DB_DATABASE` | `advisor` |
| `VITE_PORT` | `5175` |
| `FORWARD_REDIS_PORT` | `6381` |
| `FORWARD_MAILPIT_PORT` | `1029` |
| `FORWARD_MAILPIT_DASHBOARD_PORT` | `8026` |

```bash
# SSL cert
cd docker/certs && mkcert -cert-file cert.pem -key-file key.pem advisor.test
cp cert.pem ~/docker/proxy/certs/advisor.test.crt
cp key.pem ~/docker/proxy/certs/advisor.test.key
docker exec nginx-proxy nginx -s reload
cd ../..

# Windows hosts (run in WSL)
echo "127.0.0.1 advisor.test" | sudo tee -a /mnt/c/Windows/System32/drivers/etc/hosts
echo "127.0.0.1 advisor.test" | sudo tee -a /etc/hosts

# Database
docker exec mysql mysql -uroot -ppassword -e "CREATE DATABASE IF NOT EXISTS advisor CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Install and start
composer install
npm install
php artisan key:generate
./vendor/bin/sail up -d
./vendor/bin/sail artisan migrate
```

---

### larablocks.com
```bash
git clone git@github.com:YOUR_ORG/larablocks.com.git ~/code/larablocks.com
cd ~/code/larablocks.com
cp .env.example .env
```

Update `.env` with the correct values for this project (check the existing `.env.example`).

```bash
# SSL cert
cd docker/certs && mkcert -cert-file cert.pem -key-file key.pem larablocks.test
cp cert.pem ~/docker/proxy/certs/larablocks.test.crt
cp key.pem ~/docker/proxy/certs/larablocks.test.key
docker exec nginx-proxy nginx -s reload
cd ../..

# Windows hosts
echo "127.0.0.1 larablocks.test" | sudo tee -a /mnt/c/Windows/System32/drivers/etc/hosts
echo "127.0.0.1 larablocks.test" | sudo tee -a /etc/hosts

# Databases (this project uses two)
docker exec mysql mysql -uroot -ppassword -e "CREATE DATABASE IF NOT EXISTS larablocks CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
docker exec mysql mysql -uroot -ppassword -e "CREATE DATABASE IF NOT EXISTS vueblocks CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Install and start
composer install
npm install
php artisan key:generate
./vendor/bin/sail up -d
./vendor/bin/sail artisan migrate
```

---

## 4. Restore Database Data

If migrating from another machine, import the database dump:

```bash
docker exec -i mysql mysql -uroot -ppassword < ~/all-databases.sql
```

To create a dump on the source machine:

```bash
docker exec mysql mysqldump -uroot -ppassword --all-databases > ~/all-databases.sql
```

---

## 5. Verify

For each project:
- [ ] App loads at its `.test` domain with a trusted cert
- [ ] Login/register works
- [ ] `npm run dev` starts Vite with no cert warnings
- [ ] Tests pass: `./vendor/bin/sail artisan test --compact`
