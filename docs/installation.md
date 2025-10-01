# ⚙️ Guia de Instalação e Configuração

Este guia fornece instruções detalhadas para configurar o ambiente de desenvolvimento do Coin Vault.

## 📋 Pré-requisitos

### Software Necessário

- **Node.js**: versão 18.x ou superior ([Download](https://nodejs.org/))
- **npm** ou **yarn**: gerenciador de pacotes (incluído com Node.js)
- **Python**: versão 3.11 ou superior ([Download](https://www.python.org/))
- **PostgreSQL**: versão 15 ou superior ([Download](https://www.postgresql.org/))
- **Git**: para controle de versão ([Download](https://git-scm.com/))
- **Docker** (opcional): para ambiente containerizado ([Download](https://www.docker.com/))

### Verificando Instalações

```powershell
# Verificar Node.js
node --version  # deve retornar v18.x.x ou superior

# Verificar npm
npm --version

# Verificar Python
python --version  # deve retornar 3.11.x ou superior

# Verificar PostgreSQL
psql --version

# Verificar Docker (opcional)
docker --version
docker-compose --version
```

## 🐳 Opção 1: Instalação com Docker (Recomendado)

A forma mais rápida de começar é usar Docker. Todo o ambiente será configurado automaticamente.

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
# Entrar no container do backend
docker-compose exec backend bash

# Executar migrações
alembic upgrade head

# Sair do container
exit
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
docker-compose exec backend python manage.py shell
```

## 💻 Opção 2: Instalação Manual

### Configuração do Backend (FastAPI)

#### 1. Clone e Navegue até o Backend

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

# Se houver erro de execução, execute:
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

# Email (opcional, para futuras features)
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

#### 1. Navegue até o Frontend

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

O frontend estará disponível em: http://localhost:5173 (ou http://localhost:3000)

## 🗄️ Configuração Avançada do Banco de Dados

### Criando um Usuário Admin

```powershell
# Entre no container ou ative o venv do backend
cd backend

# Execute o script Python
python -m app.scripts.create_superuser

# Ou use o shell interativo
python
```

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

print("Admin user created!")
```

### Populando Dados de Teste

```powershell
# Execute o script de seed
python -m app.scripts.seed_database

# Ou crie dados manualmente via API
```

## 🧪 Configuração de Testes

### Backend Tests

```powershell
cd backend

# Criar banco de dados de teste
psql -U postgres
CREATE DATABASE coinvault_test;
\q

# Executar todos os testes
pytest

# Com cobertura
pytest --cov=app --cov-report=html

# Executar testes específicos
pytest tests/test_transactions.py

# Com verbose
pytest -v
```

### Frontend Tests

```powershell
cd frontend

# Executar testes unitários
npm run test

# Com cobertura
npm run test:coverage

# Testes E2E (Cypress)
npm run test:e2e

# Abrir Cypress UI
npm run cypress:open
```

## 🔧 Ferramentas de Desenvolvimento

### Linting e Formatação

**Backend:**
```powershell
# Black (formatação)
black app/

# Flake8 (linting)
flake8 app/

# isort (organizar imports)
isort app/

# mypy (type checking)
mypy app/
```

**Frontend:**
```powershell
# ESLint
npm run lint

# Prettier
npm run format

# Type checking
npm run type-check
```

### Git Hooks (Pre-commit)

```powershell
# Instalar pre-commit
pip install pre-commit

# Instalar hooks
pre-commit install

# Executar manualmente
pre-commit run --all-files
```

## 🐛 Troubleshooting

### Problema: Erro ao conectar no PostgreSQL

**Solução:**
```powershell
# Verificar se PostgreSQL está rodando
Get-Service -Name postgresql*

# Iniciar serviço
Start-Service postgresql-x64-15

# Verificar conexão
psql -U postgres -h localhost
```

### Problema: Porta já em uso

**Solução:**
```powershell
# Encontrar processo usando a porta
netstat -ano | findstr :8000

# Matar processo (substitua PID)
taskkill /PID <PID> /F

# Ou mudar a porta no .env
```

### Problema: Módulo Python não encontrado

**Solução:**
```powershell
# Verificar se venv está ativado
where python  # deve apontar para venv

# Reinstalar dependências
pip install -r requirements.txt --force-reinstall
```

### Problema: Erro CORS no Frontend

**Solução:**
```env
# Adicionar origem no backend/.env
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
```

### Problema: Migrações Alembic

**Solução:**
```powershell
# Ver histórico de migrações
alembic history

# Reverter migração
alembic downgrade -1

# Recriar banco do zero (CUIDADO: apaga dados)
alembic downgrade base
alembic upgrade head
```

## 📦 Configuração de IDEs

### Visual Studio Code

**Extensões Recomendadas:**
- Python (Microsoft)
- Pylance
- ESLint
- Prettier
- Tailwind CSS IntelliSense
- Docker
- PostgreSQL
- Thunder Client (testar API)

**settings.json:**
```json
{
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": false,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  }
}
```

### PyCharm

1. Configure o interpretador Python para o venv
2. Marque `app` como Sources Root
3. Configure o servidor de desenvolvimento
4. Enable Django support (opcional)

## 🚀 Próximos Passos

Agora que seu ambiente está configurado:

1. ✅ Explore a [Documentação da API](./API.md)
2. ✅ Leia sobre a [Arquitetura](./ARCHITECTURE.md)
3. ✅ Veja o [Guia de Contribuição](./CONTRIBUTING.md)
4. ✅ Comece a desenvolver!

## 💡 Dicas

- Use `docker-compose` para desenvolvimento rápido
- Mantenha os `.env` files **FORA** do controle de versão
- Execute testes regularmente
- Use branches para features novas
- Documente mudanças importantes

## 📞 Suporte

Problemas com a instalação?
- Abra uma [issue no GitHub](https://github.com/jvnds16/coin-vault/issues)
- Consulte a documentação oficial das tecnologias
