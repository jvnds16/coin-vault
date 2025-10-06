# Guia de Instalação e Configuração

Este guia fornece instruções detalhadas para configurar o ambiente de desenvolvimento do Coin Vault.

## Pré-requisitos

### Software Necessário

- **Node.js**: versão 19+ ou superior
- **npm** ou **yarn**: gerenciador de pacotes
- **Python**: versão 3.10+ ou superior
- **PostgreSQL**: versão 16+ ou superior
- **Git**: para controle de versão
- **Docker** (opcional): para ambiente containerizado

### Verificando Instalações

```powershell
# Verificar Node.js
node --version

# Verificar npm
npm --version

# Verificar Python
python --version

# Verificar PostgreSQL
psql --version

# Verificar Docker (opcional)
docker --version
docker-compose --version
```

## Opção 1: Instalação com Docker (Recomendado)

### 1. Clone o Repositório

```powershell
git clone https://github.com/jvnds16/coin-vault.git
cd coin-vault
```

### 2. Configure as Variáveis de Ambiente

```powershell
# Backend
Copy-Item backend\.env.example backend\.env

# Frontend
Copy-Item frontend\.env.example frontend\.env
```

### 3. Edite os Arquivos de Ambiente

**backend/.env:**
```env
# Database
DATABASE_URL=postgresql://coinvault:coinvault123@db:5432/coinvault
POSTGRES_USER=coinvault
POSTGRES_PASSWORD=coinvault123
POSTGRES_DB=coinvault

# JWT
SECRET_KEY=sua-chave-secreta-muito-segura-aqui-mude-em-producao
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# App
ENVIRONMENT=development
DEBUG=True
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
```

**frontend/.env:**
```env
VITE_API_URL=http://localhost:8000/api/v1
VITE_APP_NAME=Coin Vault
```

### 4. Inicie os Containers

```powershell
# Construir e iniciar todos os serviços
docker-compose up -d --build

# Verificar status dos containers
docker-compose ps

# Ver logs
docker-compose logs -f
```

### 5. Execute as Migrações do Banco de Dados

```powershell
# Executar migrações
docker-compose exec backend alembic upgrade head
```

### 6. Acesse a Aplicação

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:8000
- **API Documentation**: http://localhost:8000/docs
- **PostgreSQL**: localhost:5432

### Comandos Docker Úteis

```powershell
# Parar todos os containers
docker-compose stop

# Parar e remover containers
docker-compose down

# Remover containers e volumes (CUIDADO: apaga dados)
docker-compose down -v

# Reconstruir um serviço específico
docker-compose up -d --build frontend

# Ver logs de um serviço específico
docker-compose logs -f backend

# Executar comando no container
docker-compose exec backend python -m pytest
```

## Opção 2: Instalação Manual

### Configuração do Backend (FastAPI)

#### 1. Clone e Navegue ate o Backend

```powershell
git clone https://github.com/jvnds16/coin-vault.git
cd coin-vault\backend
```

#### 2. Crie um Ambiente Virtual Python

```powershell
# Criar ambiente virtual
python -m venv venv

# Ativar ambiente virtual
.\venv\Scripts\Activate.ps1

# Se houver erro de execucao, execute:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

#### 3. Instale as Dependências

```powershell
# Atualizar pip
python -m pip install --upgrade pip

# Instalar dependências
pip install -r requirements.txt

# Instalar dependências de desenvolvimento
pip install -r requirements-dev.txt
```

#### 4. Configure o Banco de Dados PostgreSQL

```powershell
# Abrir psql
psql -U postgres

# Dentro do psql, execute:
CREATE DATABASE coinvault;
CREATE USER coinvault WITH PASSWORD 'coinvault123';
GRANT ALL PRIVILEGES ON DATABASE coinvault TO coinvault;
\q
```

#### 5. Configure as Variáveis de Ambiente

```powershell
# Copiar arquivo de exemplo
Copy-Item .env.example .env

# Editar .env com suas configurações
notepad .env
```

**backend/.env:**
```env
# Database
DATABASE_URL=postgresql://coinvault:coinvault123@localhost:5432/coinvault

# JWT
SECRET_KEY=sua-chave-secreta-aqui-mude-para-algo-aleatorio
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# App
ENVIRONMENT=development
DEBUG=True
CORS_ORIGINS=http://localhost:3000,http://localhost:5173

# Email (opcional)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=seu-email@gmail.com
SMTP_PASSWORD=sua-senha
```

#### 6. Execute as Migrações

```powershell
# Criar uma migração inicial (se necessário)
alembic revision --autogenerate -m "Initial migration"

# Aplicar migrações
alembic upgrade head
```

#### 7. Inicie o Servidor Backend

```powershell
# Desenvolvimento (com hot-reload)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Ou use o script de desenvolvimento
python run.py
```

O backend estará disponível em:
- API: http://localhost:8000
- Documentação Interativa: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

### Configuração do Frontend (React)

#### 1. Navegue ate o Frontend

```powershell
# Em um novo terminal
cd coin-vault\frontend
```

#### 2. Instale as Dependências

```powershell
# Com npm
npm install

# Ou com yarn
yarn install
```

#### 3. Configure as Variáveis de Ambiente

```powershell
# Copiar arquivo de exemplo
Copy-Item .env.example .env

# Editar .env
notepad .env
```

**frontend/.env:**
```env
VITE_API_URL=http://localhost:8000/api/v1
VITE_APP_NAME=Coin Vault
VITE_APP_VERSION=1.0.0
VITE_ENABLE_ANALYTICS=false
```

#### 4. Inicie o Servidor de Desenvolvimento

```powershell
# Com npm
npm run dev

# Ou com yarn
yarn dev
```

O frontend estará disponível em: http://localhost:5173

## Configuração Avançada do Banco de Dados

### Criando um Usuário Admin

```powershell
# Entre no container ou ative o venv do backend
cd backend

# Execute o script Python
python -m app.scripts.create_superuser
```

Ou manualmente:

```python
from app.core.database import SessionLocal
from app.models.user import User
from app.core.security import get_password_hash

db = SessionLocal()

admin = User(
    email="admin@coinvault.com",
    username="admin",
    hashed_password=get_password_hash("admin123"),
    full_name="Administrator",
    is_superuser=True,
    is_active=True
)

db.add(admin)
db.commit()
db.close()

print("Admin user created successfully!")
```

### Populando Dados de Teste

```powershell
# Executar script de seed
python -m app.scripts.seed_data
```

## Testes

### Backend

```powershell
# Executar todos os testes
pytest

# Executar com cobertura
pytest --cov=app tests/

# Executar testes especificos
pytest tests/test_users.py
```

### Frontend

```powershell
# Executar testes
npm run test

# Executar com cobertura
npm run test:coverage

# Executar testes E2E
npm run test:e2e
```

## Troubleshooting

### Erro de Conexão com Banco de Dados

```powershell
# Verificar se PostgreSQL está rodando
Get-Service -Name postgresql*

# Iniciar serviço
Start-Service postgresql-x64-15
```

### Erro de Porta em Uso

```powershell
# Verificar processos na porta 8000
Get-NetTCPConnection -LocalPort 8000

# Matar processo (substitua PID)
Stop-Process -Id PID -Force
```

### Erro de Permissão no Docker

```powershell
# Executar PowerShell como Administrador e reiniciar Docker Desktop
```

## Próximos Passos

Após a instalação:

1. Acesse http://localhost:8000/docs para ver a documentação da API
2. Crie uma conta de usuário através do endpoint `/auth/register`
3. Faça login para obter um token de acesso
4. Comece a criar transações e categorias

## Links Úteis

- Documentação da API: `/docs/05-API.md`
- Arquitetura: `/docs/04-Architecture.md`
- Banco de Dados: `/docs/06-Database.md`
- Deploy: `/docs/07-Deployment.md`