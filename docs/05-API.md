# API Documentation

Documentação da API REST do Coin Vault.

## Base URL

- Development: `http://localhost:8000/api/v1`
- Production: `https://api.coinvault.com/api/v1`

## Autenticação

JWT (JSON Web Tokens). Header requerido:

```
Authorization: Bearer <token>
```

## Códigos HTTP

| Código | Descrição |
|--------|-----------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 404 | Not Found |
| 422 | Validation Error |
| 500 | Server Error |

## Endpoints

### Autenticação

#### POST /auth/register
Registrar novo usuário.

```json
{
  "email": "user@example.com",
  "username": "username",
  "password": "senha123",
  "full_name": "Nome Completo"
}
```

#### POST /auth/login
Fazer login e obter token.

```json
Request: {
  "email": "user@example.com",
  "password": "senha123"
}

Response: {
  "access_token": "...",
  "token_type": "bearer",
  "expires_in": 1800,
  "user": {...}
}
```

#### POST /auth/refresh
Renovar token de acesso.
Header: `Authorization: Bearer <token>`

#### POST /auth/logout
Fazer logout.
Header: `Authorization: Bearer <token>`

### Usuários

#### GET /users/me
Obter perfil do usuário atual.
Header: `Authorization: Bearer <token>`

#### PATCH /users/me
Atualizar perfil.

```json
{
  "full_name": "Novo Nome",
  "email": "novo@email.com"
}
```

#### POST /users/me/change-password
Alterar senha.

```json
{
  "current_password": "senha_atual",
  "new_password": "nova_senha"
}
```

### Transações

#### GET /transactions
Listar transações com filtros.

Query params:
- `skip`: offset para paginação (default: 0)
- `limit`: quantidade de registros (default: 50)
- `type`: income ou expense
- `category_id`: filtrar por categoria
- `start_date`: data inicial (YYYY-MM-DD)
- `end_date`: data final (YYYY-MM-DD)
- `search`: busca por descrição

#### GET /transactions/{id}
Obter transação por ID.

#### POST /transactions
Criar nova transação.

```json
{
  "type": "expense",
  "amount": 100.00,
  "description": "Compra no mercado",
  "date": "2025-10-01",
  "category_id": 1,
  "is_recurring": false
}
```

#### PUT /transactions/{id}
Atualizar transação existente.

#### DELETE /transactions/{id}
Deletar transação.

#### GET /transactions/stats
Estatísticas de transações.

Query params:
- `start_date`: data inicial
- `end_date`: data final

Response:
```json
{
  "total_income": 5000.00,
  "total_expense": 3000.00,
  "balance": 2000.00,
  "transactions_count": 50
}
```

### Categorias

#### GET /categories
Listar categorias.

Query params:
- `type`: income ou expense

#### GET /categories/{id}
Obter categoria por ID.

#### POST /categories
Criar nova categoria.

```json
{
  "name": "Alimentação",
  "type": "expense",
  "icon": "food",
  "color": "#FF5733"
}
```

#### PUT /categories/{id}
Atualizar categoria.

#### DELETE /categories/{id}
Deletar categoria.

### Dashboard

#### GET /dashboard/summary
Resumo geral do dashboard.

Response:
```json
{
  "current_balance": 10000.00,
  "monthly_income": 5000.00,
  "monthly_expense": 3000.00,
  "top_categories": [...]
}
```

#### GET /dashboard/cashflow
Fluxo de caixa mensal.

Query params:
- `months`: número de meses (default: 12)

## Modelos de Dados

### User
```json
{
  "id": 1,
  "email": "user@example.com",
  "username": "username",
  "full_name": "Nome Completo",
  "is_active": true,
  "created_at": "2025-01-01T00:00:00Z"
}
```

### Transaction
```json
{
  "id": 1,
  "user_id": 1,
  "category_id": 1,
  "type": "expense",
  "amount": 100.00,
  "description": "Descrição",
  "date": "2025-10-01",
  "is_recurring": false,
  "created_at": "2025-10-01T10:00:00Z"
}
```

### Category
```json
{
  "id": 1,
  "user_id": 1,
  "name": "Alimentação",
  "type": "expense",
  "icon": "food",
  "color": "#FF5733",
  "created_at": "2025-01-01T00:00:00Z"
}
```

## Tratamento de Erros

### Erro Padrão
```json
{
  "detail": "Mensagem de erro",
  "status_code": 400
}
```

### Erro de Validação (422)
```json
{
  "detail": [
    {
      "loc": ["body", "amount"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

## Recursos Adicionais

- **Rate Limiting**: 100 requisições por minuto por IP
- **Paginação**: Parâmetros `skip` e `limit`
- **Filtros**: Suporte a múltiplos filtros por endpoint
- **Ordenação**: Parâmetros `order_by` e `order` (asc/desc)

## Exemplos de Uso

### cURL

```bash
# Login
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"senha123"}'

# Criar transação
curl -X POST http://localhost:8000/api/v1/transactions \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"expense","amount":100,"description":"Compra","date":"2025-10-01","category_id":1}'
```

### JavaScript

```javascript
// Login
const response = await fetch('http://localhost:8000/api/v1/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'user@example.com', password: 'senha123' })
});
const { access_token } = await response.json();

// Listar transações
const transactions = await fetch('http://localhost:8000/api/v1/transactions', {
  headers: { 'Authorization': `Bearer ${access_token}` }
}).then(r => r.json());
```

### Python

```python
import requests

# Login
response = requests.post(
    'http://localhost:8000/api/v1/auth/login',
    json={'email': 'user@example.com', 'password': 'senha123'}
)
token = response.json()['access_token']

# Criar transação
requests.post(
    'http://localhost:8000/api/v1/transactions',
    json={
        'type': 'expense',
        'amount': 100.00,
        'description': 'Compra',
        'date': '2025-10-01',
        'category_id': 1
    },
    headers={'Authorization': f'Bearer {token}'}
)
```

## Documentação Interativa

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## Versionamento

- Versão atual: `/api/v1/`
- Breaking changes serão implementadas em: `/api/v2/`