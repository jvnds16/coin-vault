# 🚀 Guia de Deploy

Este guia cobre as diferentes estratégias de deploy do Coin Vault para ambientes de produção.

## 📋 Índice

- [Checklist Pré-Deploy](#checklist-pré-deploy)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Deploy com Docker](#deploy-com-docker)
- [Deploy em VPS](#deploy-em-vps)
- [Deploy em Cloud](#deploy-em-cloud)
- [CI/CD](#cicd)
- [Monitoramento](#monitoramento)
- [Backup](#backup)

## ✅ Checklist Pré-Deploy

Antes de fazer deploy para produção, certifique-se de:

### Segurança
- [ ] Alterar todas as secrets e keys do `.env`
- [ ] Usar senha forte para banco de dados
- [ ] Configurar HTTPS/SSL
- [ ] Configurar CORS adequadamente
- [ ] Desabilitar DEBUG mode
- [ ] Remover credenciais de teste
- [ ] Configurar rate limiting
- [ ] Revisar permissões de acesso

### Performance
- [ ] Habilitar cache (Redis)
- [ ] Configurar connection pooling
- [ ] Otimizar queries do banco
- [ ] Minificar assets do frontend
- [ ] Configurar CDN para assets estáticos
- [ ] Habilitar compressão (gzip)

### Monitoramento
- [ ] Configurar logs
- [ ] Configurar error tracking (Sentry)
- [ ] Configurar métricas (Prometheus)
- [ ] Configurar health checks
- [ ] Configurar alertas

### Backup
- [ ] Configurar backup automático do banco
- [ ] Testar restore de backup
- [ ] Configurar backup de arquivos
- [ ] Documentar processo de recovery

## 🔐 Variáveis de Ambiente

### Backend (.env.production)

```env
# Environment
ENVIRONMENT=production
DEBUG=False

# Database
DATABASE_URL=postgresql://user:strong_password@db-host:5432/coinvault
POSTGRES_USER=coinvault_prod
POSTGRES_PASSWORD=use_strong_password_here
POSTGRES_DB=coinvault_prod

# Security
SECRET_KEY=generate_a_very_long_random_secret_key_here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# CORS
CORS_ORIGINS=https://coinvault.com,https://www.coinvault.com

# Redis (Cache)
REDIS_URL=redis://redis-host:6379/0

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=noreply@coinvault.com
SMTP_PASSWORD=email_app_password
SMTP_FROM=noreply@coinvault.com

# Sentry (Error Tracking)
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id

# App
APP_NAME=Coin Vault
FRONTEND_URL=https://coinvault.com
```

### Frontend (.env.production)

```env
VITE_API_URL=https://api.coinvault.com/api/v1
VITE_APP_NAME=Coin Vault
VITE_APP_VERSION=1.0.0
VITE_ENABLE_ANALYTICS=true
VITE_SENTRY_DSN=https://your-frontend-sentry-dsn@sentry.io/project-id
```

### Gerar Secret Key Segura

```powershell
# Python
python -c "import secrets; print(secrets.token_urlsafe(64))"

# PowerShell
-join ((65..90) + (97..122) + (48..57) | Get-Random -Count 64 | ForEach-Object {[char]$_})
```

## 🐳 Deploy com Docker

### Estrutura Docker

```
coin-vault/
├── docker-compose.prod.yml
├── backend/
│   ├── Dockerfile.prod
│   └── .env.production
├── frontend/
│   ├── Dockerfile.prod
│   └── .env.production
└── nginx/
    ├── Dockerfile
    └── nginx.conf
```

### docker-compose.prod.yml

```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    container_name: coinvault_db_prod
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    networks:
      - coinvault_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: coinvault_redis_prod
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - coinvault_network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    container_name: coinvault_backend_prod
    restart: always
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    env_file:
      - ./backend/.env.production
    networks:
      - coinvault_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
      args:
        VITE_API_URL: ${VITE_API_URL}
    container_name: coinvault_frontend_prod
    restart: always
    depends_on:
      - backend
    networks:
      - coinvault_network

  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    container_name: coinvault_nginx_prod
    restart: always
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - frontend
      - backend
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    networks:
      - coinvault_network
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 10s
      retries: 3

  certbot:
    image: certbot/certbot
    container_name: coinvault_certbot
    volumes:
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  coinvault_network:
    driver: bridge
```

### Backend Dockerfile.prod

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Instalar dependências do sistema
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements
COPY requirements.txt .

# Instalar dependências Python
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código
COPY . .

# Criar usuário não-root
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Expor porta
EXPOSE 8000

# Comando de inicialização
CMD ["sh", "-c", "alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4"]
```

### Frontend Dockerfile.prod

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copiar package files
COPY package*.json ./

# Instalar dependências
RUN npm ci

# Copiar código fonte
COPY . .

# Build da aplicação
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

RUN npm run build

# Production stage
FROM nginx:alpine

# Copiar build
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuração nginx
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Nginx Configuration

```nginx
# nginx/nginx.conf
upstream backend {
    server backend:8000;
}

upstream frontend {
    server frontend:80;
}

# Redirect HTTP to HTTPS
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

# HTTPS Server
server {
    listen 443 ssl http2;
    server_name coinvault.com www.coinvault.com;

    ssl_certificate /etc/letsencrypt/live/coinvault.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/coinvault.com/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Frontend
    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend API
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # API Docs
    location /docs {
        proxy_pass http://backend;
        proxy_set_header Host $host;
    }

    # Static files cache
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        proxy_pass http://frontend;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### Deploy Steps

```powershell
# 1. Clonar repositório no servidor
git clone https://github.com/jvnds16/coin-vault.git
cd coin-vault

# 2. Configurar variáveis de ambiente
Copy-Item backend\.env.example backend\.env.production
Copy-Item frontend\.env.example frontend\.env.production
# Editar os arquivos com valores de produção

# 3. Obter certificado SSL (Let's Encrypt)
docker-compose -f docker-compose.prod.yml run --rm certbot certonly --webroot --webroot-path /var/www/certbot -d coinvault.com -d www.coinvault.com

# 4. Build e iniciar containers
docker-compose -f docker-compose.prod.yml up -d --build

# 5. Verificar status
docker-compose -f docker-compose.prod.yml ps

# 6. Ver logs
docker-compose -f docker-compose.prod.yml logs -f

# 7. Executar migrações
docker-compose -f docker-compose.prod.yml exec backend alembic upgrade head

# 8. Criar usuário admin
docker-compose -f docker-compose.prod.yml exec backend python -m app.scripts.create_superuser
```

## 🖥️ Deploy em VPS (DigitalOcean, Linode, etc)

### Requisitos Mínimos

- **CPU**: 2 cores
- **RAM**: 4 GB
- **Storage**: 40 GB SSD
- **OS**: Ubuntu 22.04 LTS

### Setup do Servidor

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Instalar Docker Compose
sudo apt install docker-compose-plugin

# Adicionar usuário ao grupo docker
sudo usermod -aG docker $USER

# Configurar firewall
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

# Instalar Git
sudo apt install git -y
```

### Deploy Automatizado

```bash
# Script de deploy
#!/bin/bash

set -e

echo "🚀 Starting deployment..."

# Pull latest changes
git pull origin main

# Build and restart containers
docker-compose -f docker-compose.prod.yml down
docker-compose -f docker-compose.prod.yml up -d --build

# Run migrations
docker-compose -f docker-compose.prod.yml exec -T backend alembic upgrade head

# Health check
sleep 10
curl -f http://localhost/api/v1/health || exit 1

echo "✅ Deployment completed successfully!"
```

## ☁️ Deploy em Cloud

### AWS (Elastic Beanstalk)

```yaml
# .ebextensions/01_environment.config
option_settings:
  aws:elasticbeanstalk:application:environment:
    ENVIRONMENT: production
    DATABASE_URL: $RDS_CONNECTION_STRING
```

### Heroku

```yaml
# Procfile
web: uvicorn app.main:app --host 0.0.0.0 --port $PORT
release: alembic upgrade head
```

```powershell
# Deploy
heroku create coinvault-api
heroku addons:create heroku-postgresql:hobby-dev
heroku config:set SECRET_KEY=your-secret-key
git push heroku main
```

### Google Cloud Run

```yaml
# cloudbuild.yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/coinvault-backend', './backend']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/coinvault-backend']
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    args:
      - 'run'
      - 'deploy'
      - 'coinvault-backend'
      - '--image=gcr.io/$PROJECT_ID/coinvault-backend'
      - '--region=us-central1'
      - '--platform=managed'
```

## 🔄 CI/CD

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          cd backend
          pip install -r requirements.txt
      
      - name: Run tests
        run: |
          cd backend
          pytest
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install frontend dependencies
        run: |
          cd frontend
          npm ci
      
      - name: Run frontend tests
        run: |
          cd frontend
          npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/coin-vault
            git pull origin main
            docker-compose -f docker-compose.prod.yml up -d --build
            docker-compose -f docker-compose.prod.yml exec -T backend alembic upgrade head
```

## 📊 Monitoramento

### Health Check Endpoint

```python
# backend/app/api/v1/endpoints/health.py
@router.get("/health")
async def health_check(db: Session = Depends(get_db)):
    try:
        # Check database
        db.execute(text("SELECT 1"))
        return {
            "status": "healthy",
            "database": "connected",
            "timestamp": datetime.utcnow()
        }
    except Exception as e:
        return {
            "status": "unhealthy",
            "error": str(e)
        }, 503
```

### Logs Centralizados

```python
# backend/app/core/logging.py
import logging
import json_logging

json_logging.init_fastapi(enable_json=True)
json_logging.init_request_instrument(app)

logger = logging.getLogger("coinvault")
logger.setLevel(logging.INFO)
```

## 💾 Backup

### Backup Automático

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Backup do banco de dados
docker-compose exec -T db pg_dump -U $POSTGRES_USER $POSTGRES_DB | gzip > $BACKUP_DIR/db_$TIMESTAMP.sql.gz

# Upload para S3 (opcional)
aws s3 cp $BACKUP_DIR/db_$TIMESTAMP.sql.gz s3://coinvault-backups/

# Manter apenas últimos 30 backups
find $BACKUP_DIR -name "db_*.sql.gz" -mtime +30 -delete

echo "Backup completed: db_$TIMESTAMP.sql.gz"
```

### Cron Job

```bash
# Adicionar ao crontab
crontab -e

# Backup diário às 2AM
0 2 * * * /var/www/coin-vault/backup.sh >> /var/log/backup.log 2>&1
```

## 🔍 Troubleshooting

### Container não inicia
```powershell
# Ver logs
docker-compose -f docker-compose.prod.yml logs backend

# Verificar configuração
docker-compose -f docker-compose.prod.yml config
```

### Erro de conexão com banco
```powershell
# Testar conexão
docker-compose -f docker-compose.prod.yml exec db psql -U $POSTGRES_USER -d $POSTGRES_DB

# Verificar health check
docker-compose -f docker-compose.prod.yml ps
```

### SSL/HTTPS não funciona
```powershell
# Renovar certificado
docker-compose -f docker-compose.prod.yml exec certbot certbot renew

# Testar nginx
docker-compose -f docker-compose.prod.yml exec nginx nginx -t
```

## 📞 Suporte

Para problemas de deploy, consulte:
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- Issues no GitHub
