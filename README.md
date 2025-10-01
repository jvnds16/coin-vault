# 💰 Coin Vault

> Sistema completo de gestão financeira pessoal com controle de receitas, despesas, investimentos e análises financeiras.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![React](https://img.shields.io/badge/React-18.x-blue.svg)](https://reactjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green.svg)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-blue.svg)](https://www.postgresql.org/)

## 📋 Sobre o Projeto

Coin Vault é uma aplicação web moderna para gerenciamento financeiro pessoal que permite aos usuários controlar suas finanças de forma intuitiva e eficiente. Com recursos avançados de análise e visualização de dados, ajuda você a tomar decisões financeiras mais inteligentes.

### ✨ Principais Funcionalidades

- 💳 **Gestão de Transações**: Registro e categorização de receitas e despesas
- 📊 **Dashboard Interativo**: Visualização de dados financeiros em tempo real
- 📈 **Análises e Relatórios**: Gráficos e insights sobre seus hábitos financeiros
- 🎯 **Metas Financeiras**: Defina e acompanhe seus objetivos
- 🔔 **Alertas**: Notificações de vencimentos e limites de gastos
- 🔐 **Segurança**: Autenticação JWT e criptografia de dados sensíveis
- 📱 **Design Responsivo**: Interface adaptável para todos os dispositivos

## 🚀 Tecnologias

### Frontend
- **React 19+** - Biblioteca para interface de usuário
- **TypeScript** - Tipagem estática para JavaScript
- **Vite** - Build tool moderna e rápida
- **TailwindCSS** - Framework CSS utility-first
- **React Query** - Gerenciamento de estado e cache de dados
- **React Router** - Navegação client-side
- **Chart.js / Recharts** - Visualização de dados

### Backend
- **FastAPI** - Framework web moderno e performático
- **Python 3.10+** - Linguagem de programação
- **SQLAlchemy** - ORM para interação com banco de dados
- **Pydantic** - Validação de dados
- **Alembic** - Migração de banco de dados
- **JWT** - Autenticação e autorização
- **Pytest** - Framework de testes

### Banco de Dados
- **PostgreSQL 15+** - Banco de dados relacional
- **Redis** (opcional) - Cache e sessões

### DevOps
- **Docker** - Containerização
- **Docker Compose** - Orquestração de containers
- **GitHub Actions** - CI/CD
- **Nginx** - Servidor web e proxy reverso

## 📦 Instalação Rápida

### Pré-requisitos

- Node.js 18+ e npm/yarn
- Python 3.11+
- PostgreSQL 15+
- Docker e Docker Compose (opcional)

### Clone o Repositório

```bash
git clone https://github.com/jvnds16/coin-vault.git
cd coin-vault
```

### Opção 1: Docker (Recomendado)

```bash
# Inicie todos os serviços
docker-compose up -d

# A aplicação estará disponível em:
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000
# API Docs: http://localhost:8000/docs
```

### Opção 2: Setup Manual

Consulte o [Guia de Instalação Completo](./docs/INSTALLATION.md) para instruções detalhadas.

## 📚 Documentação

- [📖 Arquitetura do Sistema](./docs/ARCHITECTURE.md)
- [📋 Especificações do Projeto](./docs/SPECIFICATIONS.md)
- [📐 Metodologia de Desenvolvimento](./docs/METHODOLOGY.md)
- [⚙️ Guia de Instalação](./docs/INSTALLATION.md)
- [🔌 Documentação da API](./docs/API.md)
- [🗄️ Schema do Banco de Dados](./docs/DATABASE.md)
- [🚀 Guia de Deploy](./docs/DEPLOYMENT.md)

## 🎯 Roadmap

- [x] Sistema de autenticação e autorização
- [x] CRUD de transações financeiras
- [x] Dashboard com gráficos básicos
- [ ] Sistema de categorias personalizadas
- [ ] Importação de extratos bancários (OFX/CSV)
- [ ] Aplicativo mobile (React Native)
- [ ] Integração com APIs bancárias
- [ ] Machine Learning para previsões financeiras
- [ ] Modo multi-moeda
- [ ] Compartilhamento de contas (família)

## 📝 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👤 Autor

**João Victor**

- GitHub: [@jvnds16](https://github.com/jvnds16)

---
