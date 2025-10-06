# Documentação do Banco de Dados

Documentação completa do schema do banco de dados PostgreSQL do Coin Vault.

## Visão Geral

O banco de dados é estruturado de forma relacional, utilizando PostgreSQL como SGBD.

### Características Principais

- **SGBD**: PostgreSQL 16+
- **ORM**: SQLAlchemy 2.0
- **Migrações**: Alembic
- **Encoding**: UTF-8
- **Timezone**: UTC

## Diagrama ER

```
┌─────────────────────┐
│       USERS         │
│─────────────────────│
│ PK id               │
│    email            │
│    username         │
│    hashed_password  │
│    full_name        │
│    is_active        │
│    is_superuser     │
│    created_at       │
│    updated_at       │
└──────────┬──────────┘
           │
           │ 1:N
    ┌──────┴──────┬──────────────┐
    │             │              │
┌───▼─────────┐  ┌─▼──────────────┐  ┌───▼───────────┐
│ CATEGORIES  │  │  TRANSACTIONS  │  │    BUDGETS    │
│─────────────│  │────────────────│  │───────────────│
│ PK id       │  │ PK id          │  │ PK id         │
│ FK user_id  │◄─┤ FK user_id     │  │ FK user_id    │
│    name     │  │ FK category_id │  │ FK category_id│
│    type     │  │    type        │  │    amount     │
│    icon     │  │    amount      │  │    period     │
│    color    │  │    description │  │    start_date │
│ created_at  │  │    date        │  │    end_date   │
└─────────────┘  │ is_recurring   │  │ created_at    │
                 │ created_at     │  │ updated_at    │
                 │ updated_at     │  └───────────────┘
                 └────────────────┘
```

## Tabelas

### Users

Armazena informações dos usuários do sistema.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    is_superuser BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$'),
    CONSTRAINT username_length CHECK (LENGTH(username) >= 3)
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
```

**Campos:**
- `id`: Identificador único (PK)
- `email`: Email único do usuário
- `username`: Nome de usuário único
- `hashed_password`: Senha criptografada (bcrypt)
- `full_name`: Nome completo
- `is_active`: Conta ativa/inativa
- `is_superuser`: Flag de administrador
- `created_at`: Data de criação
- `updated_at`: Data da última atualização

**Constraints:**
- Email deve ser único e válido
- Username deve ser único e ter pelo menos 3 caracteres
- Password deve ser hasheado

### Categories

Categorias de transações (receitas e despesas).

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL CHECK (type IN ('income', 'expense')),
    icon VARCHAR(50),
    color VARCHAR(7) CHECK (color ~* '^#[0-9A-F]{6}$'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_category UNIQUE (user_id, name, type)
);

CREATE INDEX idx_categories_user ON categories(user_id);
CREATE INDEX idx_categories_type ON categories(type);
```

**Campos:**
- `id`: Identificador único (PK)
- `user_id`: Referência ao usuário (FK)
- `name`: Nome da categoria
- `type`: Tipo (`income` ou `expense`)
- `icon`: Emoji ou ícone
- `color`: Cor em formato hex (#RRGGBB)
- `created_at`: Data de criação

**Constraints:**
- Cada usuário não pode ter categorias duplicadas
- Type deve ser 'income' ou 'expense'
- Color deve estar em formato hex válido

**Categorias Padrão:**
```sql
-- Despesas
INSERT INTO categories (user_id, name, type, icon, color) VALUES
(?, 'Alimentação', 'expense', 'food', '#FF5733'),
(?, 'Transporte', 'expense', 'car', '#33C1FF'),
(?, 'Moradia', 'expense', 'home', '#4CAF50'),
(?, 'Saúde', 'expense', 'health', '#E91E63'),
(?, 'Educação', 'expense', 'education', '#9C27B0'),
(?, 'Lazer', 'expense', 'entertainment', '#FF9800');

-- Receitas
INSERT INTO categories (user_id, name, type, icon, color) VALUES
(?, 'Salário', 'income', 'salary', '#4CAF50'),
(?, 'Freelance', 'income', 'work', '#2196F3'),
(?, 'Investimentos', 'income', 'investment', '#FF9800');
```

### Transactions

Registro de todas as transações financeiras.

```sql
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    type VARCHAR(20) NOT NULL CHECK (type IN ('income', 'expense')),
    amount DECIMAL(12, 2) NOT NULL CHECK (amount > 0),
    description TEXT,
    date DATE NOT NULL,
    is_recurring BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_transactions_user ON transactions(user_id);
CREATE INDEX idx_transactions_user_date ON transactions(user_id, date DESC);
CREATE INDEX idx_transactions_category ON transactions(category_id);
CREATE INDEX idx_transactions_date ON transactions(date DESC);
CREATE INDEX idx_transactions_type ON transactions(type);
```

**Campos:**
- `id`: Identificador único (PK)
- `user_id`: Referência ao usuário (FK)
- `category_id`: Referência à categoria (FK, nullable)
- `type`: Tipo (`income` ou `expense`)
- `amount`: Valor da transação (sempre positivo)
- `description`: Descrição/observação
- `date`: Data da transação
- `is_recurring`: Flag de transação recorrente
- `created_at`: Data de criação do registro
- `updated_at`: Data da última atualização

**Constraints:**
- Amount deve ser maior que zero
- Type deve ser 'income' ou 'expense'
- Se categoria for deletada, category_id vira NULL

### Budgets (Futuro)

Orçamentos e metas financeiras.

```sql
CREATE TABLE budgets (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id) ON DELETE CASCADE,
    amount DECIMAL(12, 2) NOT NULL CHECK (amount > 0),
    period VARCHAR(20) NOT NULL CHECK (period IN ('monthly', 'yearly')),
    start_date DATE NOT NULL,
    end_date DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_budgets_user ON budgets(user_id);
CREATE INDEX idx_budgets_category ON budgets(category_id);
CREATE INDEX idx_budgets_period ON budgets(period);
```

## Relacionamentos

### User → Categories (1:N)
- Um usuário pode ter várias categorias
- Categorias são deletadas quando o usuário é deletado (CASCADE)

### User → Transactions (1:N)
- Um usuário pode ter várias transações
- Transações são deletadas quando o usuário é deletado (CASCADE)

### Category → Transactions (1:N)
- Uma categoria pode ter várias transações
- Quando categoria é deletada, transactions.category_id vira NULL

### User → Budgets (1:N)
- Um usuário pode ter vários orçamentos
- Orçamentos são deletados quando o usuário é deletado (CASCADE)

### Category → Budgets (1:N)
- Uma categoria pode ter vários orçamentos
- Orçamentos são deletados quando a categoria é deletada (CASCADE)

## Índices

### Índices Primários

Todos criados automaticamente nas PKs:
- `users_pkey` em `users(id)`
- `categories_pkey` em `categories(id)`
- `transactions_pkey` em `transactions(id)`

### Índices Secundários

**Performance de Queries:**
- `idx_users_email` - Busca rápida por email (login)
- `idx_users_username` - Busca rápida por username
- `idx_transactions_user_date` - Lista de transações por usuário e data
- `idx_transactions_category` - Filtro por categoria
- `idx_categories_user` - Categorias de um usuário

**Estatísticas:**
- `idx_transactions_date` - Agregações por período
- `idx_transactions_type` - Filtro por tipo de transação

## Queries Otimizadas

### Listar Transações do Mês
```sql
SELECT t.*, c.name as category_name, c.color as category_color
FROM transactions t
LEFT JOIN categories c ON t.category_id = c.id
WHERE t.user_id = ?
  AND t.date >= DATE_TRUNC('month', CURRENT_DATE)
  AND t.date < DATE_TRUNC('month', CURRENT_DATE) + INTERVAL '1 month'
ORDER BY t.date DESC;
```

### Estatísticas Mensais
```sql
SELECT
    DATE_TRUNC('month', date) as month,
    type,
    SUM(amount) as total
FROM transactions
WHERE user_id = ?
  AND date >= CURRENT_DATE - INTERVAL '12 months'
GROUP BY month, type
ORDER BY month DESC;
```

### Top Categorias
```sql
SELECT
    c.name,
    c.color,
    SUM(t.amount) as total_spent,
    COUNT(t.id) as transaction_count
FROM transactions t
JOIN categories c ON t.category_id = c.id
WHERE t.user_id = ?
  AND t.type = 'expense'
  AND t.date >= DATE_TRUNC('month', CURRENT_DATE)
GROUP BY c.id, c.name, c.color
ORDER BY total_spent DESC
LIMIT 5;
```

### Saldo Atual
```sql
SELECT
    COALESCE(SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END), 0) as total_income,
    COALESCE(SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END), 0) as total_expense,
    COALESCE(SUM(CASE WHEN type = 'income' THEN amount ELSE -amount END), 0) as balance
FROM transactions
WHERE user_id = ?;
```

## Migrações com Alembic

### Criar Nova Migração
```bash
alembic revision --autogenerate -m "descrição da mudança"
```

### Aplicar Migrações
```bash
alembic upgrade head
```

### Reverter Migração
```bash
alembic downgrade -1
```

## Backup e Restore

### Backup
```bash
pg_dump -U coinvault -d coinvault > backup.sql
```

### Restore
```bash
psql -U coinvault -d coinvault < backup.sql
```

## Manutenção

### Vacuum e Analyze
```sql
VACUUM ANALYZE transactions;
VACUUM ANALYZE categories;
VACUUM ANALYZE users;
```

### Reindex
```sql
REINDEX TABLE transactions;
```