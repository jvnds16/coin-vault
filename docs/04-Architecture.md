# Arquitetura do Sistema

## Visão Geral

Arquitetura de três camadas (Three-Tier Architecture) separando apresentação, lógica de negócio e persistência de dados.

```
┌─────────────────────────────────────────────────────────┐
│                     FRONTEND (React)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   UI Layer   │  │  State Mgmt  │  │  API Client  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP/REST
┌────────────────────────▼────────────────────────────────┐
│                   BACKEND (FastAPI)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  API Routes  │  │   Services   │  │  Repository  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└────────────────────────┬────────────────────────────────┘
                         │ SQLAlchemy ORM
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
├── src/
│   ├── api/              # Cliente API
│   ├── components/       # Componentes reutilizáveis
│   ├── features/         # Features por domínio
│   ├── pages/            # Páginas principais
│   ├── hooks/            # Custom Hooks
│   ├── store/            # Estado global
│   ├── types/            # TypeScript types
│   └── utils/            # Utilitários
├── package.json
├── tsconfig.json
└── vite.config.ts
```

### Padrões

- **Atomic Design**: Componentes em átomos, moléculas e organismos
- **React Query**: Estado do servidor e cache
- **Context API**: Estado global
- **TypeScript**: Tipagem estática

### API Client

```typescript
const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  headers: { 'Content-Type': 'application/json' }
});

apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

## Backend Architecture

### Estrutura de Pastas

```
backend/
├── app/
│   ├── api/v1/endpoints/    # Endpoints da API
│   ├── core/                # Config e segurança
│   ├── models/              # Modelos SQLAlchemy
│   ├── schemas/             # Schemas Pydantic
│   ├── services/            # Lógica de negócio
│   ├── repositories/        # Acesso a dados
│   └── main.py
├── alembic/                 # Migrações
└── tests/
```

### Camadas

#### API Layer

```python
@router.post("/", response_model=TransactionResponse)
async def create_transaction(
    transaction: TransactionCreate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    return await transaction_service.create(db, transaction, current_user.id)
```

#### Service Layer

```python
class TransactionService:
    async def create(self, db: Session, data: TransactionCreate, user_id: int):
        if data.amount <= 0:
            raise ValueError("Amount must be positive")
        transaction = await self.repo.create(db, data, user_id)
        await self._update_account_balance(db, transaction)
        return transaction
```

#### Repository Layer

```python
class TransactionRepository:
    async def create(self, db: Session, data: TransactionCreate, user_id: int):
        transaction = Transaction(**data.dict(), user_id=user_id)
        db.add(transaction)
        await db.commit()
        return transaction
```

### Autenticação

```python
def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    user = await user_repo.get(db, payload.get("sub"))
    if not user:
        raise HTTPException(status_code=401)
    return user
```

## Database

### Schema

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL,
    icon VARCHAR(50),
    color VARCHAR(7)
);

CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    type VARCHAR(20) NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    description TEXT,
    date DATE NOT NULL,
    is_recurring BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_transactions_user_date ON transactions(user_id, date DESC);
CREATE INDEX idx_transactions_category ON transactions(category_id);
```

### Relacionamentos

- User → Transactions: One-to-Many
- User → Categories: One-to-Many
- Category → Transactions: One-to-Many

### Performance

- Indexação em colunas consultadas
- Paginação para grandes volumes
- Connection pooling
- Eager loading de relacionamentos

## Segurança

### Autenticação
- JWT para autenticação stateless
- Refresh tokens para renovação
- Bcrypt para hash de senhas

### Proteção
- HTTPS obrigatório em produção
- CORS configurado
- Rate limiting
- SQL Injection prevenido via ORM
- Validação via Pydantic e TypeScript

## Escalabilidade

### Horizontal Scaling
- Backend stateless
- Load balancer (Nginx)
- Database connection pooling

### Cache
- Redis para queries frequentes
- Cache de sessões
- Cache de dados agregados

### Otimizações
- Code splitting
- Lazy loading
- Compressão de assets
- CDN para estáticos

## Monitoramento

### Logging
- Logs estruturados (JSON)
- Níveis: DEBUG, INFO, WARNING, ERROR, CRITICAL

### Métricas
- Tempo de resposta
- Taxa de erro
- Uso de recursos

### Ferramentas
- Sentry: Error tracking
- Prometheus + Grafana: Métricas
- ELK Stack: Logs centralizados

## Testes

### Frontend
- Unit: Jest + React Testing Library
- E2E: Cypress

### Backend
- Unit: Pytest
- Integration: Pytest + banco de teste
- API: Pytest + httpx

### Cobertura
- Objetivo: maior que 80%
- Foco em lógica crítica

## CI/CD

```
build → test → lint → security_scan → deploy
```

1. Build: Compilação de assets
2. Test: Execução de testes
3. Lint: Verificação de código
4. Security: Análise de vulnerabilidades
5. Deploy: Deploy automatizado