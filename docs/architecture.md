# 🏗️ Arquitetura do Sistema

## Visão Geral

O Coin Vault é construído seguindo uma arquitetura moderna de três camadas (Three-Tier Architecture), separando claramente as responsabilidades entre apresentação, lógica de negócio e persistência de dados.

```
┌─────────────────────────────────────────────────────────┐
│                     FRONTEND (React)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   UI Layer   │  │  State Mgmt  │  │  API Client  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP/REST
                         │
┌────────────────────────▼────────────────────────────────┐
│                   BACKEND (FastAPI)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  API Routes  │  │   Services   │  │  Repository  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │ SQLAlchemy ORM
                         │
┌────────────────────────▼────────────────────────────────┐
│                DATABASE (PostgreSQL)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │    Users     │  │ Transactions │  │  Categories  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Frontend Architecture

### Estrutura de Pastas

```
frontend/
├── public/                # Arquivos estáticos
├── src/
│   ├── api/              # Cliente API e configurações
│   ├── assets/           # Imagens, fontes, ícones
│   ├── components/       # Componentes reutilizáveis
│   │   ├── common/      # Componentes genéricos (Button, Input, etc)
│   │   ├── forms/       # Formulários
│   │   └── layout/      # Layout components (Header, Sidebar, etc)
│   ├── features/        # Features organizadas por domínio
│   │   ├── auth/       # Autenticação
│   │   ├── dashboard/  # Dashboard
│   │   └── transactions/ # Transações
│   ├── hooks/          # Custom React Hooks
│   ├── pages/          # Páginas/Views principais
│   ├── routes/         # Configuração de rotas
│   ├── services/       # Lógica de negócio do frontend
│   ├── store/          # Gerenciamento de estado global
│   ├── styles/         # Estilos globais
│   ├── types/          # TypeScript types e interfaces
│   ├── utils/          # Funções utilitárias
│   ├── App.tsx         # Componente raiz
│   └── main.tsx        # Entry point
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.js
```

### Padrões e Princípios

#### 1. Componentização
- **Atomic Design**: Componentes organizados em átomos, moléculas e organismos
- **Single Responsibility**: Cada componente tem uma única responsabilidade
- **Composição sobre Herança**: Preferência por composição de componentes

#### 2. Gerenciamento de Estado
- **React Query**: Para estado do servidor (cache, sincronização)
- **Context API**: Para estado global da aplicação
- **Local State**: useState para estado local de componentes

#### 3. Roteamento
```typescript
// Exemplo de estrutura de rotas
const routes = [
  {
    path: '/',
    element: <Layout />,
    children: [
      { path: 'dashboard', element: <Dashboard /> },
      { path: 'transactions', element: <Transactions /> },
      { path: 'reports', element: <Reports /> },
    ]
  },
  {
    path: '/auth',
    element: <AuthLayout />,
    children: [
      { path: 'login', element: <Login /> },
      { path: 'register', element: <Register /> },
    ]
  }
];
```

#### 4. API Communication
```typescript
// api/client.ts
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  }
});

// Interceptors para autenticação
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

## Backend Architecture

### Estrutura de Pastas

```
backend/
├── alembic/              # Migrações do banco de dados
├── app/
│   ├── api/              # Endpoints da API
│   │   ├── v1/          # Versão 1 da API
│   │   │   ├── endpoints/
│   │   │   │   ├── auth.py
│   │   │   │   ├── transactions.py
│   │   │   │   ├── categories.py
│   │   │   │   └── users.py
│   │   │   └── api.py   # Agregador de rotas
│   │   └── deps.py      # Dependências (auth, db session)
│   ├── core/            # Configurações e segurança
│   │   ├── config.py    # Variáveis de ambiente
│   │   ├── security.py  # JWT, hash de senhas
│   │   └── database.py  # Conexão com DB
│   ├── models/          # Modelos SQLAlchemy
│   │   ├── user.py
│   │   ├── transaction.py
│   │   └── category.py
│   ├── schemas/         # Schemas Pydantic
│   │   ├── user.py
│   │   ├── transaction.py
│   │   └── category.py
│   ├── services/        # Lógica de negócio
│   │   ├── auth_service.py
│   │   ├── transaction_service.py
│   │   └── analytics_service.py
│   ├── repositories/    # Acesso a dados
│   │   ├── user_repo.py
│   │   └── transaction_repo.py
│   ├── middleware/      # Middlewares customizados
│   ├── utils/           # Funções utilitárias
│   └── main.py          # Entry point
├── tests/               # Testes
├── requirements.txt
└── pyproject.toml
```

### Camadas da Aplicação

#### 1. API Layer (Routes)
- Responsável por receber requisições HTTP
- Validação inicial de dados (via Pydantic)
- Delegação para camada de serviço
- Formatação de respostas

```python
# api/v1/endpoints/transactions.py
@router.post("/", response_model=TransactionResponse)
async def create_transaction(
    transaction: TransactionCreate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Cria uma nova transação."""
    return await transaction_service.create(db, transaction, current_user.id)
```

#### 2. Service Layer
- Contém a lógica de negócio
- Orquestra operações entre repositórios
- Validações de regras de negócio
- Transformações de dados

```python
# services/transaction_service.py
class TransactionService:
    def __init__(self, transaction_repo: TransactionRepository):
        self.repo = transaction_repo
    
    async def create(self, db: Session, data: TransactionCreate, user_id: int):
        # Validações de negócio
        if data.amount <= 0:
            raise ValueError("Amount must be positive")
        
        # Criação da transação
        transaction = await self.repo.create(db, data, user_id)
        
        # Lógica adicional (ex: atualizar saldo)
        await self._update_account_balance(db, transaction)
        
        return transaction
```

#### 3. Repository Layer
- Abstração do acesso a dados
- Operações CRUD
- Queries complexas
- Isolamento da lógica de persistência

```python
# repositories/transaction_repo.py
class TransactionRepository:
    async def create(self, db: Session, data: TransactionCreate, user_id: int):
        transaction = Transaction(**data.dict(), user_id=user_id)
        db.add(transaction)
        await db.commit()
        await db.refresh(transaction)
        return transaction
    
    async def get_by_user(self, db: Session, user_id: int, skip: int = 0, limit: int = 100):
        return db.query(Transaction)\
            .filter(Transaction.user_id == user_id)\
            .offset(skip)\
            .limit(limit)\
            .all()
```

### Autenticação e Autorização

```python
# core/security.py
def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

# api/deps.py
async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    credentials_exception = HTTPException(
        status_code=401,
        detail="Could not validate credentials"
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = await user_repo.get(db, user_id)
    if user is None:
        raise credentials_exception
    return user
```

## Database Architecture

### Schema Design

```sql
-- Usuários
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    is_superuser BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categorias
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL, -- 'income' ou 'expense'
    icon VARCHAR(50),
    color VARCHAR(7),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Transações
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    type VARCHAR(20) NOT NULL, -- 'income' ou 'expense'
    amount DECIMAL(12, 2) NOT NULL,
    description TEXT,
    date DATE NOT NULL,
    is_recurring BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Índices para performance
CREATE INDEX idx_transactions_user_date ON transactions(user_id, date DESC);
CREATE INDEX idx_transactions_category ON transactions(category_id);
CREATE INDEX idx_categories_user ON categories(user_id);
```

### Relacionamentos

- **User → Transactions**: One-to-Many
- **User → Categories**: One-to-Many
- **Category → Transactions**: One-to-Many

### Estratégias de Performance

1. **Indexação**: Índices em colunas frequentemente consultadas
2. **Paginação**: Limit/Offset para grandes volumes de dados
3. **Eager Loading**: Carregamento antecipado de relacionamentos
4. **Connection Pooling**: Pool de conexões para reutilização

## Segurança

### Autenticação
- JWT (JSON Web Tokens) para autenticação stateless
- Tokens de refresh para renovação segura
- Hash de senhas com bcrypt

### Autorização
- Verificação de ownership (usuário só acessa seus dados)
- Role-based access control (futuro)

### Proteção de Dados
- HTTPS obrigatório em produção
- CORS configurado adequadamente
- Rate limiting para prevenir abuso
- Sanitização de inputs
- SQL Injection prevenido via ORM

### Validação
- Pydantic schemas no backend
- Zod/Yup no frontend
- Validação de tipos via TypeScript

## Escalabilidade

### Horizontal Scaling
- Stateless backend permite múltiplas instâncias
- Load balancer (Nginx) distribui tráfego
- Database connection pooling

### Caching
- Redis para cache de queries frequentes
- Cache de sessões de usuário
- Cache de dados agregados (dashboards)

### Otimizações
- Code splitting no frontend
- Lazy loading de componentes
- Compressão de assets
- CDN para arquivos estáticos

## Monitoramento e Logs

### Logging
- Logs estruturados (JSON)
- Níveis: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Tracking de requisições com request_id

### Métricas
- Tempo de resposta de APIs
- Taxa de erro
- Uso de recursos
- Usuários ativos

### Ferramentas Sugeridas
- **Sentry**: Error tracking
- **Prometheus + Grafana**: Métricas e dashboards
- **ELK Stack**: Centralização de logs

## Testes

### Frontend
- **Unit Tests**: Jest + React Testing Library
- **Integration Tests**: Cypress/Playwright
- **E2E Tests**: Cypress

### Backend
- **Unit Tests**: Pytest
- **Integration Tests**: Pytest com banco de teste
- **API Tests**: Pytest + httpx

### Cobertura de Testes
- Objetivo: > 80% de cobertura
- Foco em lógica de negócio crítica
- Testes de casos extremos

## CI/CD Pipeline

```yaml
# Exemplo de workflow
build → test → lint → security_scan → deploy
```

1. **Build**: Compilação e build de assets
2. **Test**: Execução de todos os testes
3. **Lint**: Verificação de padrões de código
4. **Security Scan**: Análise de vulnerabilidades
5. **Deploy**: Deploy automatizado para staging/production

## Próximos Passos

- [ ] Implementar WebSockets para atualizações em tempo real
- [ ] Adicionar GraphQL como alternativa ao REST
- [ ] Implementar microsserviços para features específicas
- [ ] Adicionar message queue (RabbitMQ/Kafka) para processamento assíncrono
- [ ] Implementar feature flags para rollout gradual
