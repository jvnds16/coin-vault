# 📐 Metodologia

> Este documento descreve a metodologia de trabalho utilizada no desenvolvimento do Coin Vault, incluindo ambientes de trabalho, ferramentas, processos de desenvolvimento e gestão de equipe.

## Relação de Ambientes de Trabalho

Os artefatos do projeto são desenvolvidos a partir de diversas plataformas. A tabela abaixo especifica cada ambiente com sua respectiva plataforma/framework e link de acesso:

| Ambiente | Plataforma / Framework | Link de Acesso |
| -------- | ---------------------- | -------------- |
| **Cliente Web** | React.js 19+, TypeScript, Vite, TailwindCSS | [GitHub - Repositório Frontend](https://github.com/jvnds16/coin-vault/tree/main/frontend) |
| **API Backend** | FastAPI (Python 3.11+), SQLAlchemy | [GitHub - Repositório Backend](https://github.com/jvnds16/coin-vault/tree/main/backend) |
| **Banco de Dados** | PostgreSQL 15+ | Local / Railway / Supabase |
| **Containerização** | Docker, Docker Compose | [Docker Hub](https://hub.docker.com/) |
| **Controle de Versão** | Git, GitHub | [GitHub - coin-vault](https://github.com/jvnds16/coin-vault) |
| **CI/CD** | GitHub Actions | [GitHub Actions - Workflows](https://github.com/jvnds16/coin-vault/actions) |
| **Documentação** | Markdown, GitHub Pages | [GitHub - Docs](https://github.com/jvnds16/coin-vault/tree/main/docs) |
| **Gerenciamento de Projeto** | GitHub Projects, Issues | [GitHub Projects](https://github.com/jvnds16/coin-vault/projects) |
| **API Documentation** | FastAPI Swagger UI | http://localhost:8000/docs |
| **Editor de Código** | Visual Studio Code | [VS Code](https://code.visualstudio.com/) |

## Controle de Versão

A ferramenta de controle de versão adotada no projeto é o [Git](https://git-scm.com/), sendo que o [GitHub](https://github.com) é utilizado para hospedagem do repositório.

O estilo de desenvolvimento adotado é **[Trunk-Based Development](https://trunkbaseddevelopment.com/)**, privilegiando ciclos curtos de integração e evitando a criação de branches de longa duração. Este modelo promove integração contínua, reduz conflitos de merge e acelera o feedback sobre mudanças.

### Estrutura de Branches

#### Branch Principal
- **`main`**
  - Contém sempre a versão estável e testada do software
  - Cada alteração é integrada por meio de Pull Requests
  - Pull Requests passam por revisão de código e integração contínua (CI)
  - Deploy automático para produção ocorre a partir desta branch

#### Feature Branches (Curta Duração)
- Criadas a partir da `main`
- Tempo de vida máximo: 2-3 dias
- Nomeadas conforme o propósito seguindo o padrão:
  
  **Formato**: `tipo/número-descrição-curta`
  
  **Exemplos**:
  - `feature/23-dashboard-charts` - Nova funcionalidade
  - `bugfix/45-transaction-validation` - Correção de bug
  - `refactor/auth-service` - Refatoração de código
  - `docs/api-documentation` - Atualização de documentação
  - `test/transaction-service` - Adição de testes

- São removidas automaticamente após o merge

### Fluxo de Trabalho (Workflow)

#### 1. Criação de Branch

```bash
# Atualizar main local
git checkout main
git pull origin main

# Criar nova branch
git checkout -b feature/123-nova-funcionalidade
```

#### 2. Desenvolvimento

- Fazer commits pequenos e frequentes
- Seguir convenção de commits (Conventional Commits)
- Executar testes localmente antes de push

```bash
# Exemplo de commits
git commit -m "feat: adiciona validação de transações"
git commit -m "test: adiciona testes para TransactionService"
git commit -m "docs: atualiza documentação da API"
```

#### 3. Sincronização Frequente

```bash
# Manter branch atualizada com main
git fetch origin
git rebase origin/main
```

#### 4. Push e Pull Request

```bash
# Enviar alterações
git push origin feature/123-nova-funcionalidade

# Criar Pull Request no GitHub
# Seguir template de PR (descrição, checklist, screenshots)
```

#### 5. Code Review

- Pelo menos 1 aprovação necessária
- CI/CD deve passar (build, testes, lint)
- Resolver todos os comentários
- Manter discussões técnicas objetivas

#### 6. Merge

- **Squash Merge** é utilizado para manter histórico limpo
- Mensagem de merge deve seguir padrão de commit
- Branch é deletada automaticamente após merge
- Deploy automático é acionado

### Convenção de Commits

Seguimos o padrão **[Conventional Commits](https://www.conventionalcommits.org/)** para padronizar mensagens de commit:

**Formato**: `tipo(escopo): descrição`

**Tipos principais**:
- `feat`: Nova funcionalidade
- `fix`: Correção de bug
- `docs`: Documentação
- `style`: Formatação de código
- `refactor`: Refatoração sem mudança de comportamento
- `test`: Adição ou correção de testes
- `chore`: Tarefas de manutenção

**Exemplos**:
```
feat(auth): implementa login com JWT
fix(transaction): corrige cálculo de saldo
docs(readme): atualiza instruções de instalação
test(dashboard): adiciona testes de integração
refactor(api): reorganiza estrutura de endpoints
```

## Gerência de Issues

A gestão de tarefas, bugs e melhorias é realizada via **GitHub Issues**, com uso de etiquetas (labels) padronizadas para facilitar organização e priorização.

### Labels Padrão

| Label | Descrição | Cor | Uso |
|-------|-----------|-----|-----|
| `bug` | Problema em funcionalidade existente | 🔴 Vermelho | Bugs que impedem funcionamento correto |
| `feature` | Nova funcionalidade solicitada | 🟢 Verde | Novos recursos a serem implementados |
| `enhancement` | Melhoria em funcionalidade existente | 🔵 Azul | Aprimoramentos incrementais |
| `documentation` | Melhorias ou acréscimos à documentação | 📘 Azul Claro | Atualizações de docs |
| `refactor` | Refatoração de código | 🟡 Amarelo | Melhorias de arquitetura/código |
| `test` | Relacionado a testes | 🟣 Roxo | Criação ou correção de testes |
| `security` | Problema de segurança | 🟠 Laranja | Vulnerabilidades ou melhorias de segurança |
| `performance` | Problema de performance | ⚡ Amarelo | Otimizações de desempenho |
| `design` | Relacionado a UI/UX | 🎨 Rosa | Mudanças visuais e experiência |
| `priority:high` | Alta prioridade | 🔥 Vermelho Escuro | Tarefas urgentes |
| `priority:medium` | Média prioridade | 🟡 Amarelo | Tarefas importantes |
| `priority:low` | Baixa prioridade | ⚪ Cinza | Tarefas futuras |

### Fluxo de Gerenciamento de Issues

#### 1. Criação de Issue

Toda nova demanda deve ser registrada como uma issue seguindo o template apropriado:

**Template de Bug Report**:
```markdown
**Descrição do Bug**
Descrição clara e concisa do problema.

**Passos para Reproduzir**
1. Vá para '...'
2. Clique em '....'
3. Role até '....'
4. Veja o erro

**Comportamento Esperado**
O que deveria acontecer.

**Comportamento Atual**
O que está acontecendo.

**Screenshots**
Se aplicável, adicione screenshots.

**Ambiente**
- OS: [ex: Windows 11]
- Navegador: [ex: Chrome 120]
- Versão: [ex: 1.0.0]
```

**Template de Feature Request**:
```markdown
**Descrição da Funcionalidade**
Descrição clara e concisa da nova funcionalidade.

**Problema que Resolve**
Qual problema ou necessidade esta feature atende?

**Solução Proposta**
Como você imagina que esta funcionalidade deveria funcionar?

**Alternativas Consideradas**
Outras soluções que você considerou.

**Contexto Adicional**
Qualquer outra informação relevante.
```

#### 2. Triagem e Priorização

- Issues são revisadas e recebem labels apropriadas
- Prioridade é definida com base em impacto e urgência
- Issues críticas recebem `priority:high`

#### 3. Associação a Branch

Quando o desenvolvimento é iniciado:
- Criar branch referenciando o número da issue
- Exemplo: `feature/123-dashboard-charts`
- Vincular issue ao Pull Request

#### 4. Desenvolvimento

- Atualizar issue com progresso quando necessário
- Marcar issue como "In Progress" no GitHub Projects
- Referenciar issue nos commits: `feat: implementa gráfico (#123)`

#### 5. Fechamento

- Issue é fechada automaticamente ao fazer merge do PR vinculado
- Usar keywords no PR para fechar automaticamente:
  - `Closes #123`
  - `Fixes #123`
  - `Resolves #123`

### GitHub Projects

Utilizamos **GitHub Projects** (Kanban) para visualizar o fluxo de trabalho:

**Colunas**:
1. **📋 Backlog** - Issues aguardando priorização
2. **🎯 To Do** - Tarefas priorizadas para próximo sprint
3. **🚧 In Progress** - Em desenvolvimento ativo
4. **👀 In Review** - Aguardando code review
5. **✅ Done** - Concluídas e merged

**Automações**:
- Issue criada → move para Backlog
- PR aberto → move para In Review
- PR merged → move para Done
- Issue fechada → move para Done

## Metodologia de Desenvolvimento

### Abordagem Ágil

O projeto adota práticas ágeis inspiradas em **Scrum** e **Kanban**:

#### Sprints
- Duração: 2 semanas
- Planning no início (definir escopo)
- Daily standups assíncronos (GitHub Discussions)
- Review ao final (demonstração)
- Retrospective (melhorias)

#### Cerimônias

**Sprint Planning** (Início do Sprint)
- Revisar backlog
- Selecionar issues para o sprint
- Definir metas do sprint
- Estimar complexidade (Planning Poker)

**Daily Standup** (Assíncrono via GitHub)
- O que foi feito ontem?
- O que será feito hoje?
- Há algum impedimento?

**Sprint Review** (Final do Sprint)
- Demonstrar funcionalidades concluídas
- Coletar feedback
- Atualizar backlog

**Sprint Retrospective** (Final do Sprint)
- O que funcionou bem?
- O que pode melhorar?
- Ações de melhoria para próximo sprint

### Pair Programming

Encorajado para:
- Funcionalidades complexas
- Onboarding de novos membros
- Resolução de bugs críticos
- Revisão de código em tempo real

### Code Review Guidelines

**Responsabilidades do Autor**:
- ✅ Código testado localmente
- ✅ Testes automatizados incluídos
- ✅ Documentação atualizada
- ✅ CI passando sem erros
- ✅ Descrição clara no PR

**Responsabilidades do Revisor**:
- 🔍 Verificar lógica e arquitetura
- 🔍 Conferir testes adequados
- 🔍 Avaliar legibilidade do código
- 🔍 Identificar possíveis bugs
- 🔍 Sugerir melhorias

**Tempo de Resposta**:
- Code review deve ser feito em até 24h
- PRs bloqueantes têm prioridade

## Ferramentas de Desenvolvimento

### Frontend

| Ferramenta | Propósito | Configuração |
|------------|-----------|--------------|
| **ESLint** | Linting de código | `.eslintrc.js` |
| **Prettier** | Formatação automática | `.prettierrc` |
| **TypeScript** | Tipagem estática | `tsconfig.json` |
| **Vite** | Build tool | `vite.config.ts` |
| **Vitest** | Testes unitários | `vitest.config.ts` |
| **Cypress** | Testes E2E | `cypress.config.ts` |
| **React Testing Library** | Testes de componentes | - |

### Backend

| Ferramenta | Propósito | Configuração |
|------------|-----------|--------------|
| **Black** | Formatação de código | `pyproject.toml` |
| **Ruff** | Linting rápido | `ruff.toml` |
| **mypy** | Type checking | `mypy.ini` |
| **pytest** | Framework de testes | `pytest.ini` |
| **coverage** | Cobertura de testes | `.coveragerc` |
| **pre-commit** | Hooks pré-commit | `.pre-commit-config.yaml` |
| **Alembic** | Migrações de DB | `alembic.ini` |

### DevOps

| Ferramenta | Propósito |
|------------|-----------|
| **Docker** | Containerização |
| **Docker Compose** | Orquestração local |
| **GitHub Actions** | CI/CD |
| **Dependabot** | Atualização de dependências |

## Qualidade de Código

### Padrões de Código

**Frontend (TypeScript/React)**:
- Seguir [Airbnb Style Guide](https://github.com/airbnb/javascript)
- Componentes funcionais com Hooks
- Props tipadas com TypeScript
- Evitar `any`, preferir tipos específicos

**Backend (Python)**:
- Seguir [PEP 8](https://pep8.org/)
- Type hints em todas as funções
- Docstrings em classes e funções públicas
- Maximum line length: 100 caracteres

### Testes

**Cobertura Mínima**: 80%

**Pirâmide de Testes**:
```
       /\
      /E2E\         10% - Testes End-to-End
     /------\
    /Integr.\      20% - Testes de Integração
   /----------\
  /   Unit     \   70% - Testes Unitários
 /--------------\
```

**Testes Obrigatórios**:
- ✅ Lógica de negócio crítica
- ✅ Cálculos financeiros
- ✅ Autenticação e autorização
- ✅ Validações de dados
- ✅ Casos extremos (edge cases)

### Continuous Integration

**Pipeline CI (GitHub Actions)**:

```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  frontend:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Setup Node.js
      - Install dependencies
      - Run ESLint
      - Run TypeScript check
      - Run tests
      - Build application
  
  backend:
    runs-on: ubuntu-latest
    steps:
      - Checkout code
      - Setup Python
      - Install dependencies
      - Run Ruff
      - Run mypy
      - Run tests with coverage
      - Check coverage threshold
  
  security:
    runs-on: ubuntu-latest
    steps:
      - Dependency vulnerability scan
      - SAST (Static Analysis)
```

**Requisitos para Merge**:
- ✅ Todos os checks do CI passando
- ✅ Cobertura de testes ≥ 80%
- ✅ Sem vulnerabilidades críticas
- ✅ Pelo menos 1 aprovação em code review

## Documentação

### Níveis de Documentação

1. **Código** - Comentários inline e docstrings
2. **API** - Swagger/OpenAPI gerado automaticamente
3. **Arquitetura** - Diagramas e decisões técnicas
4. **Usuário** - Guias e tutoriais

### Manutenção de Docs

- Documentação atualizada junto com código
- README.md deve estar sempre atual
- Changelog mantido para cada release
- ADRs (Architecture Decision Records) para decisões importantes

## Segurança

### Práticas de Segurança

- 🔒 Nunca commitar secrets (usar `.env` e `.gitignore`)
- 🔒 Dependências auditadas regularmente
- 🔒 HTTPS obrigatório em produção
- 🔒 Input validation em todas as camadas
- 🔒 Rate limiting em APIs
- 🔒 Logs sem informações sensíveis

### Ferramentas de Segurança

- **Dependabot** - Alertas de vulnerabilidades
- **GitHub Secret Scanning** - Detecta secrets commitados
- **OWASP Dependency Check** - Análise de dependências
- **Snyk** - Monitoramento contínuo de segurança

## Comunicação

### Canais

- **GitHub Issues** - Discussões técnicas sobre features/bugs
- **GitHub Discussions** - Perguntas, ideias, anúncios
- **Pull Requests** - Code review e discussões de implementação
- **Discord/Slack** (opcional) - Comunicação em tempo real

### Boas Práticas

- Comunicação clara e objetiva
- Documentar decisões importantes
- Manter discussões nos lugares apropriados
- Ser respeitoso e construtivo

## Versionamento Semântico

Seguimos [Semantic Versioning 2.0.0](https://semver.org/):

**Formato**: `MAJOR.MINOR.PATCH`

- **MAJOR** - Mudanças incompatíveis na API
- **MINOR** - Novas funcionalidades compatíveis
- **PATCH** - Correções de bugs compatíveis

**Exemplos**:
- `1.0.0` - Release inicial
- `1.1.0` - Nova funcionalidade (ex: relatórios PDF)
- `1.1.1` - Correção de bug
- `2.0.0` - Breaking change (ex: mudança na estrutura da API)

## Releases

### Ciclo de Release

1. **Feature Freeze** - Parar de adicionar novas features
2. **Bug Fixing** - Foco em correções
3. **Testing** - Testes finais e QA
4. **Release Candidate** - Versão candidata (rc)
5. **Release** - Publicação oficial
6. **Hotfixes** - Correções críticas pós-release

### Changelog

Mantemos um `CHANGELOG.md` seguindo [Keep a Changelog](https://keepachangelog.com/):

```markdown
## [1.1.0] - 2025-10-15

### Added
- Dashboard com gráficos interativos
- Exportação de relatórios em PDF
- Filtro avançado de transações

### Changed
- Melhorado desempenho da API de transações
- Atualizada interface do formulário de categorias

### Fixed
- Corrigido cálculo de saldo mensal
- Corrigido bug de paginação no histórico

### Security
- Atualizada dependência com vulnerabilidade (CVE-2025-12345)
```

---

**Documento criado em**: 01/10/2025  
**Versão**: 1.0  
**Autor**: João Victor  
**Última atualização**: 01/10/2025
