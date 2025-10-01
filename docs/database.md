# 🗄️ Documentação do Banco de Dados

Documentação completa do schema do banco de dados PostgreSQL do Coin Vault.

## 📋 Visão Geral

O banco de dados é estruturado de forma relacional, utilizando PostgreSQL como SGBD. A arquitetura foi projetada para ser escalável, eficiente e manter a integridade dos dados.

### Características Principais

- **SGBD**: PostgreSQL 15+
- **ORM**: SQLAlchemy 2.0
- **Migrações**: Alembic
- **Encoding**: UTF-8
- **Timezone**: UTC

## 🏗️ Diagrama ER

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
           │
    ┌──────┴───────┬──────────────────────┐
    │              │                      │
    │              │                      │
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

## 📊 Tabelas

### Users

Armazena informações dos usuários do sistema.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE NOT NULL,
    is_superuser BOOLEAN DEFAULT FALSE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
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
    user_id INTEGER NOT NULL,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL,
    icon VARCHAR(50),
    color VARCHAR(7),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_categories_user 
        FOREIGN KEY (user_id) 
        REFERENCES users(id) 
        ON DELETE CASCADE,
    CONSTRAINT category_type_check 
        CHECK (type IN ('income', 'expense')),
    CONSTRAINT color_format 
        CHECK (color ~* '^#[0-9A-Fa-f]{6}$'),
    CONSTRAINT unique_category_per_user 
        UNIQUE(user_id, name, type)
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
(?, 'Alimentação', 'expense', '🍔', '#FF5733'),
(?, 'Transporte', 'expense', '🚗', '#33C1FF'),
(?, 'Moradia', 'expense', '🏠', '#4CAF50'),
(?, 'Saúde', 'expense', '🏥', '#E91E63'),
(?, 'Educação', 'expense', '📚', '#9C27B0'),
(?, 'Lazer', 'expense', '🎮', '#FF9800');

-- Receitas
INSERT INTO categories (user_id, name, type, icon, color) VALUES
(?, 'Salário', 'income', '💰', '#4CAF50'),
(?, 'Freelance', 'income', '💼', '#2196F3'),
(?, 'Investimentos', 'income', '📈', '#FF9800');
```

### Transactions

Registro de todas as transações financeiras.

```sql
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    category_id INTEGER,
    type VARCHAR(20) NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    description TEXT,
    date DATE NOT NULL,
    is_recurring BOOLEAN DEFAULT FALSE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_transactions_user 
        FOREIGN KEY (user_id) 
        REFERENCES users(id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_transactions_category 
        FOREIGN KEY (category_id) 
        REFERENCES categories(id) 
        ON DELETE SET NULL,
    CONSTRAINT transaction_type_check 
        CHECK (type IN ('income', 'expense')),
    CONSTRAINT amount_positive 
        CHECK (amount > 0)
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
    user_id INTEGER NOT NULL,
    category_id INTEGER,
    name VARCHAR(255) NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    period VARCHAR(20) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE,
    is_active BOOLEAN DEFAULT TRUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_budgets_user 
        FOREIGN KEY (user_id) 
        REFERENCES users(id) 
        ON DELETE CASCADE,
    CONSTRAINT fk_budgets_category 
        FOREIGN KEY (category_id) 
        REFERENCES categories(id) 
        ON DELETE CASCADE,
    CONSTRAINT budget_period_check 
        CHECK (period IN ('daily', 'weekly', 'monthly', 'yearly')),
    CONSTRAINT budget_amount_positive 
        CHECK (amount > 0),
    CONSTRAINT budget_dates_valid 
        CHECK (end_date IS NULL OR end_date >= start_date)
);

CREATE INDEX idx_budgets_user ON budgets(user_id);
CREATE INDEX idx_budgets_category ON budgets(category_id);
CREATE INDEX idx_budgets_period ON budgets(period);
```

## 🔑 Relacionamentos

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

## 📈 Índices

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

## 🎯 Queries Otimizadas

### Listar Transações do Mês

```sql
SELECT 
    t.id,
    t.type,
    t.amount,
    t.description,
    t.date,
    c.name as category_name,
    c.icon as category_icon,
    c.color as category_color
FROM transactions t
LEFT JOIN categories c ON t.category_id = c.id
WHERE t.user_id = ?
    AND t.date >= DATE_TRUNC('month', CURRENT_DATE)
    AND t.date < DATE_TRUNC('month', CURRENT_DATE) + INTERVAL '1 month'
ORDER BY t.date DESC, t.created_at DESC
LIMIT 50;
```

### Total por Categoria

```sql
SELECT 
    c.name,
    c.icon,
    c.color,
    SUM(t.amount) as total,
    COUNT(t.id) as transaction_count
FROM categories c
INNER JOIN transactions t ON c.id = t.category_id
WHERE c.user_id = ?
    AND t.date BETWEEN ? AND ?
    AND t.type = 'expense'
GROUP BY c.id, c.name, c.icon, c.color
ORDER BY total DESC;
```

### Saldo Atual

```sql
SELECT 
    COALESCE(SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END), 0) as total_income,
    COALESCE(SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END), 0) as total_expense,
    COALESCE(
        SUM(CASE WHEN type = 'income' THEN amount ELSE -amount END), 
        0
    ) as balance
FROM transactions
WHERE user_id = ?;
```

### Fluxo de Caixa Mensal

```sql
SELECT 
    DATE_TRUNC('month', date) as month,
    SUM(CASE WHEN type = 'income' THEN amount ELSE 0 END) as income,
    SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END) as expense,
    SUM(CASE WHEN type = 'income' THEN amount ELSE -amount END) as balance
FROM transactions
WHERE user_id = ?
    AND date >= CURRENT_DATE - INTERVAL '12 months'
GROUP BY DATE_TRUNC('month', date)
ORDER BY month DESC;
```

## 🔄 Migrações com Alembic

### Criar Nova Migração

```powershell
# Após modificar models
alembic revision --autogenerate -m "Add budgets table"

# Editar arquivo gerado em alembic/versions/
# Revisar upgrade() e downgrade()
```

### Aplicar Migrações

```powershell
# Aplicar todas pendentes
alembic upgrade head

# Aplicar migração específica
alembic upgrade <revision>

# Reverter uma migração
alembic downgrade -1

# Reverter todas
alembic downgrade base

# Ver histórico
alembic history

# Ver migração atual
alembic current
```

### Exemplo de Migração

```python
"""Add budgets table

Revision ID: abc123def456
Revises: previous_revision
Create Date: 2025-10-01 10:00:00.000000
"""
from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = 'abc123def456'
down_revision = 'previous_revision'
branch_labels = None
depends_on = None

def upgrade():
    op.create_table(
        'budgets',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('user_id', sa.Integer(), nullable=False),
        sa.Column('category_id', sa.Integer(), nullable=True),
        sa.Column('name', sa.String(255), nullable=False),
        sa.Column('amount', sa.Numeric(12, 2), nullable=False),
        sa.Column('period', sa.String(20), nullable=False),
        sa.Column('start_date', sa.Date(), nullable=False),
        sa.Column('end_date', sa.Date(), nullable=True),
        sa.Column('is_active', sa.Boolean(), default=True, nullable=False),
        sa.Column('created_at', sa.DateTime(timezone=True), 
                  server_default=sa.text('now()'), nullable=False),
        sa.Column('updated_at', sa.DateTime(timezone=True), 
                  server_default=sa.text('now()'), nullable=False),
        sa.PrimaryKeyConstraint('id'),
        sa.ForeignKeyConstraint(['user_id'], ['users.id'], ondelete='CASCADE'),
        sa.ForeignKeyConstraint(['category_id'], ['categories.id'], ondelete='CASCADE')
    )
    op.create_index('idx_budgets_user', 'budgets', ['user_id'])
    op.create_index('idx_budgets_category', 'budgets', ['category_id'])

def downgrade():
    op.drop_index('idx_budgets_category')
    op.drop_index('idx_budgets_user')
    op.drop_table('budgets')
```

## 🔒 Segurança

### Senhas

```python
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# Hash
hashed = pwd_context.hash("senha123")

# Verificar
pwd_context.verify("senha123", hashed)  # True
```

### SQL Injection Prevention

SQLAlchemy ORM previne SQL injection automaticamente:

```python
# ✅ Seguro (parametrizado)
db.query(Transaction).filter(Transaction.user_id == user_id).all()

# ❌ Inseguro (nunca fazer)
db.execute(f"SELECT * FROM transactions WHERE user_id = {user_id}")
```

## 🗂️ Backup e Restore

### Backup

```powershell
# Backup completo
pg_dump -U coinvault -d coinvault > backup_$(date +%Y%m%d).sql

# Backup apenas schema
pg_dump -U coinvault -d coinvault --schema-only > schema.sql

# Backup apenas dados
pg_dump -U coinvault -d coinvault --data-only > data.sql

# Backup de tabela específica
pg_dump -U coinvault -d coinvault -t transactions > transactions.sql
```

### Restore

```powershell
# Restaurar backup
psql -U coinvault -d coinvault < backup.sql

# Criar DB e restaurar
createdb -U postgres coinvault_restored
psql -U postgres -d coinvault_restored < backup.sql
```

## 📊 Manutenção

### Vacuum e Analyze

```sql
-- Liberar espaço e atualizar estatísticas
VACUUM ANALYZE transactions;

-- Vacuum completo (requer lock)
VACUUM FULL transactions;

-- Apenas atualizar estatísticas
ANALYZE transactions;
```

### Verificar Tamanho das Tabelas

```sql
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### Verificar Índices Não Utilizados

```sql
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

## 🔮 Melhorias Futuras

- [ ] Particionamento de `transactions` por data
- [ ] Tabela de audit log para rastreabilidade
- [ ] Soft delete em vez de hard delete
- [ ] Tabela de tags para transações
- [ ] Tabela de attachments (recibos, notas fiscais)
- [ ] Tabela de accounts (múltiplas contas bancárias)
- [ ] Tabela de goals (metas financeiras)
- [ ] Full-text search em descriptions

## 📞 Referências

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/)
- [Alembic Documentation](https://alembic.sqlalchemy.org/)
- [Database Design Best Practices](https://www.postgresql.org/docs/current/ddl-constraints.html)
