# Especificações

Esta seção detalha as especificações do "Coin Vault" a partir da perspectiva do usuário, transformando a visão do produto em requisitos claros para desenvolvimento.

## Personas

**Persona 1: Ana, a Universitária Atarefada**
- **Idade**: 21 anos
- **Ocupação**: Estudante universitária
- **Necessidades**: Controlar gastos mensais, dividir despesas com colegas, economia de tempo
- **Frustrações**: Apps complicados, tempo escasso para gerenciar finanças

**Persona 2: Rodrigo, o Jovem Profissional**
- **Idade**: 28 anos
- **Ocupação**: Analista de TI
- **Necessidades**: Organizar orçamento do casal, análises visuais, controle detalhado
- **Frustrações**: Falta de visibilidade sobre gastos, dificuldade para dividir despesas

**Persona 3: Maria, a Mãe de Família**
- **Idade**: 35 anos
- **Ocupação**: Professora
- **Necessidades**: Organizar orçamento familiar, identificar economia, planejamento
- **Frustrações**: Gastos inesperados, dificuldade para categorizar despesas

**Persona 4: Lucas, o Empreendedor**
- **Idade**: 32 anos
- **Ocupação**: Dono de negócio próprio
- **Necessidades**: Separar finanças pessoais e empresariais, controle de fluxo de caixa
- **Frustrações**: Mistura de contas, falta de visão clara do negócio

## Histórias de Usuários

Esta seção mapeia as necessidades dos usuários em histórias acionáveis alinhadas aos requisitos funcionais da primeira versão.

| ID | EU COMO... `PERSONA` | QUERO/PRECISO ... `FUNCIONALIDADE` | PARA ... `MOTIVO/VALOR` | RF Relacionado |
|----|---------------------|-----------------------------------|------------------------|----------------|
| **US-001** | Todos os usuários | Criar uma conta no sistema informando email e senha | Ter acesso personalizado às minhas despesas | RF-001 |
| **US-002** | Todos os usuários | Fazer login no sistema com email e senha | Acessar minhas despesas de forma segura | RF-001 |
| **US-003** | Ana, a Universitária | Cadastrar uma despesa rapidamente informando data, tipo, descrição e valor | Manter controle financeiro atualizado sem perder tempo | RF-002, RF-003 |
| **US-004** | Maria, a Mãe de Família | Ver todas as minhas despesas cadastradas em uma tabela organizada | Ter visão clara de todos os gastos registrados | RF-004 |
| **US-005** | Rodrigo, o Jovem Profissional | Filtrar despesas por mês e tipo | Analisar gastos específicos de um período ou categoria | RF-005 |
| **US-006** | Ana, a Universitária | Buscar despesas por descrição (ex: "restaurante") | Encontrar rapidamente gastos específicos | RF-005 |
| **US-007** | Lucas, o Empreendedor | Filtrar despesas por faixa de valor | Identificar gastos grandes ou pequenos facilmente | RF-005 |
| **US-008** | Todos os usuários | Excluir despesas cadastradas incorretamente | Manter registro financeiro livre de erros e duplicações | RF-006 |
| **US-009** | Maria, a Mãe de Família | Receber confirmação quando cadastro ou excluo uma despesa | Ter certeza de que a operação foi realizada com sucesso | RF-007 |
| **US-010** | Ana, a Universitária | Ser notificado quando ocorre um erro ao cadastrar/excluir despesa | Saber que preciso tentar novamente ou corrigir algo | RF-007 |
| **US-011** | Rodrigo, o Jovem Profissional | Acessar o sistema do meu celular com interface adaptada | Registrar despesas em qualquer lugar de forma confortável | RF-008 |
| **US-012** | Todos os usuários | Usar o sistema em tablet ou desktop com layout otimizado | Ter melhor visualização e produtividade em telas maiores | RF-008 |

## Requisitos Funcionais

Esta seção apresenta os requisitos funcionais priorizados para a primeira versão do Coin Vault, focados em funcionalidades essenciais de controle de despesas.

| ID | Descrição | Prioridade | Status |
|:---|:----------|:-----------|:-------|
| **RF-001** | O sistema deve permitir cadastro e autenticação de usuários por email e senha | **ALTA** | Planejado |
| **RF-002** | O sistema deve permitir o cadastro de novas despesas, informando ano, mês, dia, tipo, descrição e valor, salvando-as no banco de dados PostgreSQL | **ALTA** | Planejado |
| **RF-003** | O sistema deve armazenar as despesas em um banco de dados relacional (PostgreSQL) por meio de uma API desenvolvida em FastAPI | **ALTA** | Planejado |
| **RF-004** | O sistema deve permitir a consulta das despesas cadastradas, exibindo-as em uma tabela no frontend (React) | **ALTA** | Planejado |
| **RF-005** | O sistema deve oferecer filtros de pesquisa por ano, mês, dia, tipo, descrição e valor nas despesas cadastradas | **ALTA** | Planejado |
| **RF-006** | O sistema deve permitir a exclusão de despesas listadas na tabela de consulta | **MÉDIA** | Planejado |
| **RF-007** | O sistema deve exibir mensagens de sucesso ou erro ao registrar, consultar ou excluir uma despesa | **MÉDIA** | Planejado |
| **RF-008** | O sistema deve possuir uma interface responsiva, adaptando-se a diferentes tamanhos de tela e dispositivos | **MÉDIA** | Planejado |

## Requisitos Não Funcionais

| ID | Categoria | Descrição | Prioridade | Métrica de Verificação |
|:---|:----------|:----------|:-----------|:----------------------|
| **RNF-001** | Desempenho | Tempo de resposta de APIs ≤ 500ms (95th percentile) em ambiente de desenvolvimento | **ALTA** | Medição via logs ou APM |
| **RNF-002** | Usabilidade | Interface responsiva (mobile-first) com breakpoints definidos | **ALTA** | Testes visuais em múltiplos dispositivos |
| **RNF-003** | Compatibilidade | Suporte a Chrome 90+, Firefox 88+, Safari 14+, Edge 90+ | **ALTA** | Testes manuais/automatizados |
| **RNF-004** | Desempenho | Carregamento inicial da aplicação ≤ 3 segundos em rede 3G | **ALTA** | Lighthouse Performance Score ≥ 90 |
| **RNF-005** | Confiabilidade | Validação de dados em frontend e backend (defesa em profundidade) | **ALTA** | Testes de validação e2e |
| **RNF-006** | Manutenibilidade | Código organizado seguindo padrões (PEP 8, Airbnb Style Guide) | **ALTA** | Linters (Ruff, ESLint) sem erros |
| **RNF-007** | Segurança | Sanitização de inputs para prevenir SQL Injection e XSS | **ALTA** | Uso de ORM parametrizado, escape HTML |
| **RNF-008** | Portabilidade | Aplicação deve rodar em contêineres Docker | **MÉDIA** | Docker Compose up sem erros |

## Matriz de Rastreabilidade

Esta matriz relaciona os Requisitos Funcionais (RF) com os Requisitos Não Funcionais (RNF) e as Histórias de Usuário (US).

| ID | RF-001 | RF-002 | RF-003 | RF-004 | RF-005 | RF-006 | RF-007 | RF-008 | RNF-001 | RNF-002 | RNF-003 | RNF-004 | RNF-005 | RNF-006 | RNF-007 | RNF-008 |
|----|--------|--------|--------|--------|--------|--------|--------|--------|---------|---------|---------|---------|---------|---------|---------|---------|
| **RF-001** | - | X | | | | | | | X | X | X | X | X | X | X | |
| **RF-002** | X | - | X | | | | | | X | X | X | X | X | X | X | |
| **RF-003** | | X | - | X | | | | | X | | X | | X | X | X | X |
| **RF-004** | X | | X | - | X | | | | X | X | X | X | | X | X | |
| **RF-005** | X | | | X | - | | | | X | X | X | | X | X | X | |
| **RF-006** | X | | | X | | - | | | X | X | X | | X | X | X | |
| **RF-007** | | X | | X | X | X | - | | | X | X | | | X | | |
| **RF-008** | | | | | | | | - | | X | X | X | | X | | |
| **RNF-001** | X | X | X | X | X | X | | | - | | | X | X | | | |
| **RNF-002** | X | X | | X | X | X | X | X | | - | X | X | | | | |
| **RNF-003** | X | X | X | X | X | X | X | X | | X | - | X | | | | |
| **RNF-004** | X | X | | X | | | | X | X | X | X | - | | | | |
| **RNF-005** | X | X | X | | X | X | | | X | | | | - | X | X | |
| **RNF-006** | X | X | X | X | X | X | X | X | | | | | X | - | | X |
| **RNF-007** | X | X | X | X | X | X | | | | | | | X | | - | |
| **RNF-008** | | | X | | | | | | | | | | | X | | - |

**Legenda**: X indica que o requisito da linha está relacionado/depende do requisito da coluna