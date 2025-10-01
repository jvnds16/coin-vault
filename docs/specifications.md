# 📖 Especificações - Coin Vault

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

| EU COMO... `PERSONA` | QUERO/PRECISO ... `FUNCIONALIDADE` | PARA ... `MOTIVO/VALOR` |
|---------------------|-----------------------------------|------------------------|
| Ana, a Universitária | Adicionar uma despesa rapidamente | Manter meu controle financeiro atualizado em tempo real |
| Ana, a Universitária | Ver um resumo dos meus gastos por categoria | Identificar onde estou gastando mais e fazer ajustes |
| Rodrigo, o Jovem Profissional | Gerar gráficos visuais | Analisar a porcentagem de gastos em cada categoria |
| Rodrigo, o Jovem Profissional | Criar grupos de despesas compartilhadas | Dividir e rastrear gastos com minha namorada |
| Maria, a Mãe de Família | Categorizar despesas da família | Organizar orçamento doméstico e identificar economias |
| Todos os usuários | Editar uma transação já registrada | Corrigir erros e manter dados precisos |
| Lucas, o Empreendedor | Registrar receitas e despesas separadamente | Ter visão clara do fluxo de caixa do negócio |
| Todos os usuários | Excluir transações duplicadas | Manter registro financeiro livre de erros |
| Usuário | Visualizar dashboard com resumo financeiro | Ter visão geral rápida da situação financeira |
| Usuário | Filtrar transações por período e categoria | Analisar gastos específicos |
| Usuário | Criar categorias personalizadas | Organizar finanças conforme minhas necessidades |
| Administrador | Gerenciar permissões de usuários | Garantir segurança e acesso adequado |

## Requisitos Funcionais

| ID | Descrição | Prioridade | Responsável |
|:---|:----------|:-----------|:------------|
| **RF-001** | O sistema deve permitir cadastro e autenticação de usuários por email e senha | **ALTA** | - |
| **RF-002** | O sistema deve permitir criar, listar, atualizar e deletar transações (CRUD completo) | **ALTA** | - |
| **RF-003** | O sistema deve gerar gráficos visuais (pizza, linha, barra) para análise financeira | **ALTA** | - |
| **RF-004** | O sistema deve oferecer filtros por categoria, período e tipo de transação | **ALTA** | - |
| **RF-005** | O sistema deve permitir criar, editar e excluir categorias personalizadas | **ALTA** | - |
| **RF-006** | O sistema deve exibir dashboard com cards de saldo, receitas e despesas do mês | **ALTA** | - |
| **RF-007** | O sistema deve mostrar histórico completo de transações com paginação | **ALTA** | - |
| **RF-008** | O sistema deve permitir recuperação de senha via email | **MÉDIA** | - |
| **RF-009** | O sistema deve permitir edição de perfil (nome, email, foto) | **MÉDIA** | - |
| **RF-010** | O sistema deve gerar relatórios exportáveis (PDF/Excel) | **MÉDIA** | - |
| **RF-011** | O sistema deve permitir comparação de períodos financeiros | **MÉDIA** | - |
| **RF-012** | O sistema deve oferecer busca textual em transações | **BAIXA** | - |
| **RF-013** | O sistema deve permitir importação de transações via CSV/OFX | **BAIXA** | - |
| **RF-014** | O sistema deve enviar alertas de limites de gastos configuráveis | **BAIXA** | - |

## Requisitos Não Funcionais

| ID | Descrição | Prioridade |
|:---|:----------|:-----------|
| **RNF-001** | Senhas devem ser armazenadas com criptografia bcrypt | **ALTA** |
| **RNF-002** | Sistema deve usar HTTPS para todas as comunicações | **ALTA** |
| **RNF-003** | Interface deve ser responsiva (mobile-first) | **ALTA** |
| **RNF-004** | Compatibilidade com navegadores modernos (Chrome 90+, Firefox 88+, Safari 14+) | **ALTA** |
| **RNF-005** | Tempo de resposta de APIs < 1 segundo (95th percentile) | **ALTA** |
| **RNF-006** | Disponibilidade do sistema > 99.5% (uptime) | **ALTA** |
| **RNF-007** | Interface intuitiva com tempo de aprendizado < 10 minutos | **MÉDIA** |
| **RNF-008** | Carregamento inicial da aplicação < 3 segundos | **MÉDIA** |
| **RNF-009** | Conformidade com LGPD/GDPR para proteção de dados | **MÉDIA** |
| **RNF-010** | Suporte a dispositivos móveis iOS 14+ e Android 10+ | **MÉDIA** |

## Restrições

| ID | Restrição |
|:---|:----------|
| **01** | O projeto deve ser entregue até o final do semestre letivo |
| **02** | Deve utilizar PostgreSQL como banco de dados relacional |
| **03** | Backend deve ser desenvolvido em Python com FastAPI |
| **04** | Frontend deve ser desenvolvido em React com TypeScript |
| **05** | Orçamento limitado a recursos de infraestrutura gratuitos/baixo custo |

## Matriz de Rastreabilidade

| ID | RF-001 | RF-002 | RF-003 | RF-004 | RF-005 | RF-006 | RNF-001 | RNF-002 | RNF-003 | RNF-005 | RNF-007 |
|----|--------|--------|--------|--------|--------|--------|---------|---------|---------|---------|---------|
| **RF-001** | - | | | | | | X | X | X | X | X |
| **RF-002** | | - | | | | | X | X | X | X | X |
| **RF-003** | | | - | | | X | | X | X | X | X |
| **RF-004** | | X | | - | | | | X | X | X | X |
| **RF-005** | | X | | | - | | | X | X | X | X |
| **RF-006** | X | X | X | | | - | | X | X | X | X |
| **RNF-001** | X | X | | | | | - | X | | X | |
| **RNF-002** | X | X | X | X | X | X | X | - | | X | |
| **RNF-003** | X | X | X | X | X | X | | | - | X | X |
| **RNF-005** | X | X | X | X | X | X | | | X | - | X |
| **RNF-007** | X | X | X | X | X | X | | | X | X | - |

**Legenda**: X indica que o requisito da linha está relacionado/depende do requisito da coluna

---

**Documento criado em**: 01/10/2025  
**Versão**: 1.0  
**Autor**: João Victor  
**Última atualização**: 01/10/2025
