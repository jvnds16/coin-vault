# Metodologia

> Metodologia de trabalho para desenvolvimento do Coin Vault: ambientes, ferramentas, processos e gestão.

## Stack Tecnológica

| Camada | Tecnologias |
|--------|-------------|
| **Frontend** | React 19+, TypeScript, Vite, TailwindCSS |
| **Backend** | FastAPI (Python 3.11+), SQLAlchemy |
| **Banco de Dados** | PostgreSQL 15+ |
| **Infraestrutura** | Docker, GitHub Actions, Railway/Supabase |
| **Gestão** | Git, GitHub Projects, Issues |

## Requisitos Funcionais

**Alta Prioridade**: RF-001 (Cadastro despesas), RF-002 (Persistência PostgreSQL), RF-003 (Consulta/tabela), RF-004 (Filtros)
**Média Prioridade**: RF-005 (Exclusão), RF-006 (Mensagens), RF-007 (Responsividade)

**Critérios**: Validação, HTTP status corretos, migrações, paginação, ≤2s carregamento, modais confirmação
**Fora do escopo**: Autenticação, relatórios, multi-moeda

## Controle de Versão

**Modelo**: Trunk-Based Development (branches ≤ 2-3 dias)
**Branches**: `main` (estável, protegida), `tipo/número-descrição` (features)
**Workflow**: Update main → branch → commits (Conventional Commits) → rebase → PR → review + CI → squash merge
**Conventional Commits**: `tipo(escopo): descrição` | Tipos: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

## Gestão de Issues

**Ferramenta**: GitHub Issues + Projects (Kanban)
**Labels**: `bug`, `feature`, `enhancement`, `docs`, `refactor`, `test`, `security`, `performance`, `priority:high/medium/low`
**Fluxo**: Criar (template) → Triar (labels) → Desenvolver → Referenciar commits → Fechar automático (`Closes #123`)
**Kanban**: 📋 Backlog → 🎯 To Do → 🚧 In Progress → 👀 In Review → ✅ Done

## Metodologia Ágil

**Sprints**: 2 semanas (Planning, Daily assíncrono, Review, Retrospective)
**Pair Programming**: Features complexas, onboarding, bugs críticos
**Code Review**: Autor (✅ testes, CI, docs) | Revisor (🔍 lógica, testes, bugs) | SLA 24h

## Implementação

**Arquitetura**: DTOs (Pydantic) ≠ ORM, camada serviço, middleware exceções, Decimal monetário, React Query
**Endpoints**: POST `/despesas` (201), GET `/despesas` (paginação, filtros), DELETE `/despesas/{id}` (204)
**BD**: Tabela `despesas`, índices (ano, mês, tipo), Alembic
**Frontend**: Formulários validados, DataTable responsivo (Tailwind), toasts, cards mobile

## Ferramentas

**Frontend**: ESLint, Prettier, TypeScript, Vite, Vitest, Cypress, React Testing Library

**Backend**: Black, Ruff, mypy, pytest, coverage, pre-commit, Alembic

**DevOps**: Docker, Docker Compose, GitHub Actions, Dependabot

## Qualidade e Testes

**Padrões**: Frontend (Airbnb, TypeScript), Backend (PEP 8, type hints, docstrings)
**Cobertura**: ≥ 80% (70% unit, 20% integração, 10% E2E)
**CI**: Lint, type check, testes + coverage, build, security scan
**Merge**: CI verde, cobertura ≥ 80%, sem vulnerabilidades críticas, ≥1 aprovação

## Documentação, Segurança e Comunicação

**Docs**: Código (docstrings), API (Swagger), Arquitetura (ADRs), Usuário (guias) | Atualização contínua
**Segurança**: 🔒 Sem secrets (`.env`), HTTPS, input validation, rate limiting | Dependabot, Secret Scanning, OWASP, Snyk
**Comunicação**: GitHub Issues (técnico), Discussions (geral), PRs (review) | Clareza, objetividade, respeito

## Versionamento e Releases

**Semantic Versioning**: `MAJOR.MINOR.PATCH` (breaking.feature.bugfix)
**Ciclo**: Feature Freeze → Bug Fixing → Testing → RC → Release → Hotfixes
**Changelog**: `CHANGELOG.md` (Keep a Changelog)

## Rastreabilidade dos RFs

| RF | Artefatos | Testes | Métrica DoD |
|----|-----------|--------|-------------|
| 001 | POST /despesas, ORM, Migração | Unit, Integração, E2E | HTTP 201 + persistência |
| 002 | Conexão DB, Alembic, Repositório | Integração (sessão, rollback) | Migração sem erro |
| 003 | GET /despesas | Integração, E2E (tabela) | Render lista |
| 004 | Query params | Unit (parser), Integração | Filtragem correta |
| 005 | DELETE /despesas/{id} | Integração, E2E (UI) | HTTP 204 + remoção |
| 006 | Middleware erros, Toasts | UI, Serviço | Mensagem adequada |
| 007 | Layout responsivo | Visual, Lighthouse | Score mobile ≥ 90 |

---