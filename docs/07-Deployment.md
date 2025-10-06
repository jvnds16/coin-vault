# Guia de Deploy

Este guia cobre as diferentes estrategias de deploy do Coin Vault para ambientes de producao.

## Checklist Pre-Deploy

### Segurança
- Alterar todas as secrets e keys do `.env`
- Usar senha forte para banco de dados
- Configurar HTTPS/SSL
- Configurar CORS adequadamente
- Desabilitar DEBUG mode
- Configurar rate limiting

### Performance
- Habilitar cache (Redis)
- Configurar connection pooling
- Otimizar queries do banco
- Minificar assets do frontend
- Habilitar compressao (gzip)

### Monitoramento
- Configurar logs
- Configurar error tracking (Sentry)
- Configurar health checks
- Configurar alertas

### Backup
- Configurar backup automatico do banco
- Testar restore de backup

## Variáveis de Ambiente

### Backend (.env.production)

```env
ENVIRONMENT=production
DEBUG=False

DATABASE_URL=postgresql://user:strong_password@db-host:5432/coinvault
POSTGRES_USER=coinvault_prod
POSTGRES_PASSWORD=use_strong_password_here
POSTGRES_DB=coinvault_prod

SECRET_KEY=generate_a_very_long_random_secret_key_here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

CORS_ORIGINS=https://coinvault.com,https://www.coinvault.com

REDIS_URL=redis://redis-host:6379/0

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@coinvault.com
SMTP_PASSWORD=email_app_password

SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
```

### Frontend (.env.production)

```env
VITE_API_URL=https://api.coinvault.com/api/v1
VITE_APP_NAME=Coin Vault
VITE_APP_VERSION=1.0.0
VITE_SENTRY_DSN=https://your-frontend-sentry-dsn@sentry.io/project-id
```

### Gerar Secret Key

```powershell
python -c "import secrets; print(secrets.token_urlsafe(64))"
```

## Deploy com Docker

### docker-compose.prod.yml

```yaml
version: '3.8'

services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

  redis:
    image: redis:7-alpine
    restart: always

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    env_file:
      - ./backend/.env.production
    depends_on:
      - db
      - redis
    restart: always

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
      args:
        VITE_API_URL: ${VITE_API_URL}
    restart: always

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certbot/conf:/etc/letsencrypt:ro
    depends_on:
      - backend
      - frontend
    restart: always

volumes:
  postgres_data:
```

### Backend Dockerfile.prod

```dockerfile
FROM python:3.10-slim

WORKDIR /app

RUN apt-get update && apt-get install -y gcc postgresql-client \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

CMD ["sh", "-c", "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4"]
```

### Frontend Dockerfile.prod

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Nginx Configuration

```nginx
upstream backend {
    server backend:8000;
}

server {
    listen 80;
    server_name coinvault.com www.coinvault.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name coinvault.com www.coinvault.com;

    ssl_certificate /etc/letsencrypt/live/coinvault.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/coinvault.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;

    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }
}
```

### Deploy Steps

```powershell
# 1. Clonar repositorio
git clone https://github.com/jvnds16/coin-vault.git
cd coin-vault

# 2. Configurar variaveis de ambiente
Copy-Item backend\.env.example backend\.env.production
Copy-Item frontend\.env.example frontend\.env.production

# 3. Obter certificado SSL
docker-compose -f docker-compose.prod.yml run --rm certbot certonly \
  --webroot --webroot-path /var/www/certbot \
  -d coinvault.com -d www.coinvault.com

# 4. Build e iniciar
docker-compose -f docker-compose.prod.yml up -d --build

# 5. Executar migracoes
docker-compose -f docker-compose.prod.yml exec backend alembic upgrade head

# 6. Verificar status
docker-compose -f docker-compose.prod.yml ps
docker-compose -f docker-compose.prod.yml logs -f
```

## Deploy em VPS

### Requisitos Mínimos

- CPU: 2 cores
- RAM: 4 GB
- Storage: 40 GB SSD
- OS: Ubuntu 22.04 LTS

### Setup do Servidor

```bash
sudo apt update && sudo apt upgrade -y

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

sudo apt install docker-compose-plugin

sudo usermod -aG docker $USER

sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

sudo apt install git -y
```

## Backup Automático

### Script de Backup

```bash
#!/bin/bash

BACKUP_DIR="/backups/coinvault"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

docker-compose -f docker-compose.prod.yml exec -T db \
  pg_dump -U coinvault coinvault | gzip > $BACKUP_DIR/db_backup_$DATE.sql.gz

find $BACKUP_DIR -name "db_backup_*.sql.gz" -mtime +7 -delete

echo "Backup concluído: $DATE"
```

### Configurar Cron

```bash
crontab -e

# Backup diario as 2AM
0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1
```

## Monitoramento

### Health Check Endpoint

```python
@app.get("/health")
async def health_check():
    return {"status": "healthy", "timestamp": datetime.utcnow()}
```

### Script de Monitoramento

```bash
#!/bin/bash

API_URL="https://coinvault.com/api/health"
response=$(curl -s -o /dev/null -w "%{http_code}" $API_URL)

if [ $response != "200" ]; then
    echo "API down! Status code: $response"
fi
```

## Atualizações

### Deploy de Nova Versão

```powershell
git pull origin main

docker-compose -f docker-compose.prod.yml up -d --build

docker-compose -f docker-compose.prod.yml exec backend alembic upgrade head

docker-compose -f docker-compose.prod.yml logs -f --tail=100
```

## Restauração de Backup

```bash
# Parar containers
docker-compose -f docker-compose.prod.yml down

# Restaurar backup
gunzip < backup.sql.gz | docker-compose -f docker-compose.prod.yml exec -T db \
  psql -U coinvault coinvault

# Reiniciar
docker-compose -f docker-compose.prod.yml up -d
```