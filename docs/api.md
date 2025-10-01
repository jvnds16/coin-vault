# 🔌 API Documentation

Documentação completa da API REST do Coin Vault.

## Base URL

```
Development: http://localhost:8000/api/v1
Production: https://api.coinvault.com/api/v1
```

## Autenticação

A API usa JWT (JSON Web Tokens) para autenticação. Todas as rotas protegidas requerem um token válido no header.

### Header de Autenticação

```http
Authorization: Bearer <seu_token_jwt>
```

### Obter Token

```http
POST /auth/login
Content-Type: application/json

{
  "email": "usuario@example.com",
  "password": "senha123"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 1800
}
```

## Códigos de Status HTTP

| Código | Descrição |
|--------|-----------|
| 200 | OK - Requisição bem-sucedida |
| 201 | Created - Recurso criado com sucesso |
| 204 | No Content - Requisição bem-sucedida sem conteúdo |
| 400 | Bad Request - Dados inválidos |
| 401 | Unauthorized - Não autenticado |
| 403 | Forbidden - Sem permissão |
| 404 | Not Found - Recurso não encontrado |
| 422 | Unprocessable Entity - Validação falhou |
| 500 | Internal Server Error - Erro no servidor |

## Endpoints

### 🔐 Autenticação

#### Registrar Novo Usuário

```http
POST /auth/register
Content-Type: application/json

{
  "email": "novo@example.com",
  "username": "novousuario",
  "password": "senha123",
  "full_name": "Nome Completo"
}
```

**Response (201):**
```json
{
  "id": 1,
  "email": "novo@example.com",
  "username": "novousuario",
  "full_name": "Nome Completo",
  "is_active": true,
  "created_at": "2025-10-01T10:00:00Z"
}
```

#### Login

```http
POST /auth/login
Content-Type: application/json

{
  "email": "usuario@example.com",
  "password": "senha123"
}
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 1800,
  "user": {
    "id": 1,
    "email": "usuario@example.com",
    "username": "usuario",
    "full_name": "Nome Completo"
  }
}
```

#### Renovar Token

```http
POST /auth/refresh
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "access_token": "novo_token_jwt",
  "token_type": "bearer",
  "expires_in": 1800
}
```

#### Logout

```http
POST /auth/logout
Authorization: Bearer <token>
```

**Response (204):** No content

### 👤 Usuários

#### Obter Perfil Atual

```http
GET /users/me
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "email": "usuario@example.com",
  "username": "usuario",
  "full_name": "Nome Completo",
  "is_active": true,
  "is_superuser": false,
  "created_at": "2025-01-01T10:00:00Z",
  "updated_at": "2025-10-01T10:00:00Z"
}
```

#### Atualizar Perfil

```http
PATCH /users/me
Authorization: Bearer <token>
Content-Type: application/json

{
  "full_name": "Novo Nome",
  "email": "novoemail@example.com"
}
```

**Response (200):**
```json
{
  "id": 1,
  "email": "novoemail@example.com",
  "username": "usuario",
  "full_name": "Novo Nome",
  "is_active": true,
  "updated_at": "2025-10-01T11:00:00Z"
}
```

#### Alterar Senha

```http
POST /users/me/change-password
Authorization: Bearer <token>
Content-Type: application/json

{
  "current_password": "senha_antiga",
  "new_password": "senha_nova"
}
```

**Response (200):**
```json
{
  "message": "Password updated successfully"
}
```

### 💰 Transações

#### Listar Transações

```http
GET /transactions?skip=0&limit=50&type=expense&start_date=2025-01-01&end_date=2025-12-31
Authorization: Bearer <token>
```

**Query Parameters:**
- `skip` (int, default: 0): Número de registros para pular
- `limit` (int, default: 50): Número máximo de registros
- `type` (string, optional): Filtrar por tipo (`income`, `expense`)
- `category_id` (int, optional): Filtrar por categoria
- `start_date` (date, optional): Data inicial (YYYY-MM-DD)
- `end_date` (date, optional): Data final (YYYY-MM-DD)
- `search` (string, optional): Buscar na descrição

**Response (200):**
```json
{
  "total": 150,
  "items": [
    {
      "id": 1,
      "type": "expense",
      "amount": 150.00,
      "description": "Supermercado",
      "date": "2025-10-01",
      "category": {
        "id": 5,
        "name": "Alimentação",
        "icon": "🍔",
        "color": "#FF5733"
      },
      "is_recurring": false,
      "created_at": "2025-10-01T10:00:00Z",
      "updated_at": "2025-10-01T10:00:00Z"
    }
  ],
  "page": 1,
  "pages": 3
}
```

#### Obter Transação por ID

```http
GET /transactions/{transaction_id}
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "type": "expense",
  "amount": 150.00,
  "description": "Supermercado",
  "date": "2025-10-01",
  "category_id": 5,
  "is_recurring": false,
  "created_at": "2025-10-01T10:00:00Z",
  "updated_at": "2025-10-01T10:00:00Z"
}
```

#### Criar Transação

```http
POST /transactions
Authorization: Bearer <token>
Content-Type: application/json

{
  "type": "expense",
  "amount": 150.00,
  "description": "Supermercado",
  "date": "2025-10-01",
  "category_id": 5,
  "is_recurring": false
}
```

**Response (201):**
```json
{
  "id": 100,
  "type": "expense",
  "amount": 150.00,
  "description": "Supermercado",
  "date": "2025-10-01",
  "category_id": 5,
  "is_recurring": false,
  "created_at": "2025-10-01T12:00:00Z",
  "updated_at": "2025-10-01T12:00:00Z"
}
```

#### Atualizar Transação

```http
PUT /transactions/{transaction_id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "amount": 175.50,
  "description": "Supermercado - Compra mensal"
}
```

**Response (200):**
```json
{
  "id": 1,
  "type": "expense",
  "amount": 175.50,
  "description": "Supermercado - Compra mensal",
  "date": "2025-10-01",
  "category_id": 5,
  "is_recurring": false,
  "updated_at": "2025-10-01T13:00:00Z"
}
```

#### Deletar Transação

```http
DELETE /transactions/{transaction_id}
Authorization: Bearer <token>
```

**Response (204):** No content

#### Estatísticas de Transações

```http
GET /transactions/stats?start_date=2025-01-01&end_date=2025-12-31
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "period": {
    "start_date": "2025-01-01",
    "end_date": "2025-12-31"
  },
  "total_income": 15000.00,
  "total_expense": 8500.00,
  "balance": 6500.00,
  "transaction_count": 150,
  "avg_transaction": 156.67,
  "by_category": [
    {
      "category": "Alimentação",
      "total": 2500.00,
      "count": 45,
      "percentage": 29.41
    }
  ],
  "by_month": [
    {
      "month": "2025-01",
      "income": 5000.00,
      "expense": 3000.00,
      "balance": 2000.00
    }
  ]
}
```

### 📁 Categorias

#### Listar Categorias

```http
GET /categories?type=expense
Authorization: Bearer <token>
```

**Query Parameters:**
- `type` (string, optional): Filtrar por tipo (`income`, `expense`)

**Response (200):**
```json
[
  {
    "id": 1,
    "name": "Alimentação",
    "type": "expense",
    "icon": "🍔",
    "color": "#FF5733",
    "transaction_count": 45,
    "total_amount": 2500.00,
    "created_at": "2025-01-01T10:00:00Z"
  },
  {
    "id": 2,
    "name": "Transporte",
    "type": "expense",
    "icon": "🚗",
    "color": "#33C1FF",
    "transaction_count": 30,
    "total_amount": 1200.00,
    "created_at": "2025-01-01T10:00:00Z"
  }
]
```

#### Obter Categoria por ID

```http
GET /categories/{category_id}
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "name": "Alimentação",
  "type": "expense",
  "icon": "🍔",
  "color": "#FF5733",
  "created_at": "2025-01-01T10:00:00Z"
}
```

#### Criar Categoria

```http
POST /categories
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Educação",
  "type": "expense",
  "icon": "📚",
  "color": "#4CAF50"
}
```

**Response (201):**
```json
{
  "id": 15,
  "name": "Educação",
  "type": "expense",
  "icon": "📚",
  "color": "#4CAF50",
  "created_at": "2025-10-01T12:00:00Z"
}
```

#### Atualizar Categoria

```http
PUT /categories/{category_id}
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Educação e Cursos",
  "color": "#2196F3"
}
```

**Response (200):**
```json
{
  "id": 15,
  "name": "Educação e Cursos",
  "type": "expense",
  "icon": "📚",
  "color": "#2196F3",
  "created_at": "2025-10-01T12:00:00Z"
}
```

#### Deletar Categoria

```http
DELETE /categories/{category_id}
Authorization: Bearer <token>
```

**Response (204):** No content

### 📊 Dashboard

#### Resumo Geral

```http
GET /dashboard/summary
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "current_balance": 6500.00,
  "total_income_month": 5000.00,
  "total_expense_month": 3000.00,
  "balance_month": 2000.00,
  "income_vs_last_month": 5.5,
  "expense_vs_last_month": -3.2,
  "recent_transactions": [],
  "top_categories": [],
  "upcoming_bills": []
}
```

#### Gráfico de Fluxo de Caixa

```http
GET /dashboard/cashflow?months=12
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "months": [
    {
      "month": "2025-01",
      "income": 5000.00,
      "expense": 3000.00,
      "balance": 2000.00
    }
  ]
}
```

## Modelos de Dados

### User

```typescript
interface User {
  id: number;
  email: string;
  username: string;
  full_name: string;
  is_active: boolean;
  is_superuser: boolean;
  created_at: string;
  updated_at: string;
}
```

### Transaction

```typescript
interface Transaction {
  id: number;
  user_id: number;
  category_id: number | null;
  type: "income" | "expense";
  amount: number;
  description: string;
  date: string;  // YYYY-MM-DD
  is_recurring: boolean;
  created_at: string;
  updated_at: string;
}
```

### Category

```typescript
interface Category {
  id: number;
  user_id: number;
  name: string;
  type: "income" | "expense";
  icon: string;
  color: string;  // hex color
  created_at: string;
}
```

## Tratamento de Erros

### Formato Padrão de Erro

```json
{
  "detail": "Error message",
  "status_code": 400
}
```

### Erros de Validação (422)

```json
{
  "detail": [
    {
      "loc": ["body", "email"],
      "msg": "value is not a valid email address",
      "type": "value_error.email"
    }
  ]
}
```

## Rate Limiting

- **Limite**: 100 requisições por minuto por IP
- **Header de resposta**: `X-RateLimit-Remaining`
- **Erro 429**: Too Many Requests

## Paginação

Endpoints que retornam listas suportam paginação:

```http
GET /transactions?skip=0&limit=50
```

**Resposta inclui:**
```json
{
  "total": 150,
  "items": [...],
  "page": 1,
  "pages": 3,
  "skip": 0,
  "limit": 50
}
```

## Filtros e Ordenação

### Filtros Comuns

- `type`: Filtrar por tipo
- `category_id`: Filtrar por categoria
- `start_date` / `end_date`: Filtrar por período
- `search`: Busca textual

### Ordenação

```http
GET /transactions?order_by=date&order=desc
```

## Webhooks (Futuro)

Planejado para futuras versões:
- Notificações de transações
- Alertas de limite de gastos
- Notificações de vencimentos

## API Interativa

Acesse a documentação interativa (Swagger UI):
- Development: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## Versionamento

A API segue versionamento semântico. A versão atual está no path:
- `/api/v1/...`

Mudanças breaking serão feitas em novas versões (`/api/v2/`).

## Exemplos de Uso

### cURL

```bash
# Login
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"senha123"}'

# Criar transação
curl -X POST http://localhost:8000/api/v1/transactions \
  -H "Authorization: Bearer seu_token_aqui" \
  -H "Content-Type: application/json" \
  -d '{"type":"expense","amount":100.00,"description":"Café","date":"2025-10-01","category_id":1}'
```

### JavaScript/Fetch

```javascript
// Login
const response = await fetch('http://localhost:8000/api/v1/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'senha123'
  })
});
const { access_token } = await response.json();

// Listar transações
const transactions = await fetch('http://localhost:8000/api/v1/transactions', {
  headers: { 'Authorization': `Bearer ${access_token}` }
}).then(r => r.json());
```

### Python/Requests

```python
import requests

# Login
response = requests.post(
    'http://localhost:8000/api/v1/auth/login',
    json={'email': 'user@example.com', 'password': 'senha123'}
)
token = response.json()['access_token']

# Criar transação
headers = {'Authorization': f'Bearer {token}'}
data = {
    'type': 'expense',
    'amount': 100.00,
    'description': 'Café',
    'date': '2025-10-01',
    'category_id': 1
}
response = requests.post(
    'http://localhost:8000/api/v1/transactions',
    json=data,
    headers=headers
)
```

## Suporte

Para questões sobre a API:
- Documentação interativa: `/docs`
- Issues no GitHub
- Email: dev@coinvault.com
