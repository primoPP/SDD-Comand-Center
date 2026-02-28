# Centralização de Agentes SDD - Especificação

## Sumário

Centralizar os agentes, skills e prompts do SDD Bootstrap para que equipes não precisem instalar a configuração completa em cada repositório. A feature suporta três estratégias: instalação a nível de usuário (`~/.copilot/`), a nível de organização GitHub (`.github-private`), e inicialização leve por repositório (apenas contexto). O repositório fonte de centralização é [primoPP/SDD-Comand-Center](https://github.com/primoPP/SDD-Comand-Center).

## Contexto de Negócio

O SDD Bootstrap v5.0.1 (framework de Specification-Driven Development) atualmente requer instalação completa em cada repositório via `make -f sdd.mk install`. Isso copia **8 arquivos de agentes**, **14 diretórios de skills**, **3+ arquivos de prompts**, diretórios de specs e configuração `.sdd/` para DENTRO de cada repo. Em equipes com muitos repositórios, isso gera:

- Duplicação massiva de arquivos idênticos em dezenas de repos
- Dificuldade de manter todos os repos sincronizados na mesma versão
- Overhead de PR reviews para atualizar agentes que são idênticos entre repos
- Barreira de entrada para novos repos (processo de setup longo)

A solução é centralizar os artefatos compartilhados (agentes, skills, prompts) em um único local, mantendo apenas o contexto específico do projeto em cada repo. O VS Code já possui mecanismos nativos para descoberta de agentes em múltiplos níveis que podem ser aproveitados.

### Fonte do Requisito
- **Confluence**: [Comece aqui - SDD Bootstrap Startup](https://picpay.atlassian.net/wiki/spaces/BPJ/pages/6033539279/Comece+aqui+SDD+Bootstrap+Startup)
- **Repositório Fork**: [primoPP/SDD-Comand-Center](https://github.com/primoPP/SDD-Comand-Center) (fork de microsoft/vscode-copilot-chat)

---

## Descobertas Técnicas - Mecanismos de Agentes do VS Code

### Três Níveis de Descoberta de Agentes no VS Code

Pesquisa realizada no código-fonte do vscode-copilot-chat revela **três níveis** de descoberta de agentes customizados, todos já implementados:

#### 1. Nível Workspace (Atual do SDD)

| Item | Detalhe |
|------|---------|
| **Caminho** | `.github/agents/*.agent.md` |
| **Descoberta** | Automática pelo VS Code ao abrir workspace |
| **Precedência** | Maior — workspace sempre vence |
| **Limitação** | Requer cópia dos arquivos em cada repo |

#### 2. Nível Usuário (~/.copilot/)

| Item | Detalhe |
|------|---------|
| **Caminho** | `~/.copilot/skills/`, `~/.copilot/agents/` |
| **Configuração VS Code** | `chat.instructionsFilesLocations` e `chat.agentSkillsLocations` |
| **Descoberta** | Via settings do VS Code — qualquer path pode ser adicionado |
| **Precedência** | Menor que workspace, maior que org |
| **Vantagem** | Funciona offline, sem dependência de GitHub org |

**Arquivos-chave no codebase:**
- [src/platform/customInstructions/common/customInstructionsService.ts](src/platform/customInstructions/common/customInstructionsService.ts) — Serviço core de descoberta, processa `chat.instructionsFilesLocations`
- [src/platform/customInstructions/common/promptTypes.ts](src/platform/customInstructions/common/promptTypes.ts) — Define constantes: `PERSONAL_SKILL_FOLDERS = ['.copilot/skills', '.claude/skills']`

**Configuração VS Code settings.json:**
```json
{
  "chat.instructionsFilesLocations": {
    "~/.copilot/": true
  },
  "chat.agentSkillsLocations": [
    "~/.copilot/skills/"
  ]
}
```

#### 3. Nível Organização GitHub

| Item | Detalhe |
|------|---------|
| **Repositório** | `<org>/.github-private` (repo privado especial) |
| **Polling** | A cada 5 minutos pelo VS Code |
| **Cache** | Agentes ficam em cache local do VS Code |
| **Habilitação** | `github.copilot.chat.organizationCustomAgents.enabled` (default: `true`) |
| **Precedência** | Menor — workspace e user vencem |

**Arquivos-chave no codebase:**
- [src/extension/agents/vscode-node/githubOrgCustomAgentProvider.ts](src/extension/agents/vscode-node/githubOrgCustomAgentProvider.ts) — Provider que faz polling de agentes na org via `octoKitService.getCustomAgents(orgId, repoName, { includeSources: ['org', 'enterprise'] })`
- [src/extension/agents/vscode-node/githubOrgInstructionsProvider.ts](src/extension/agents/vscode-node/githubOrgInstructionsProvider.ts) — Provedor de instruções organizacionais (polling a cada 2 min)
- [src/extension/agents/vscode-node/githubOrgChatResourcesService.ts](src/extension/agents/vscode-node/githubOrgChatResourcesService.ts) — Serviço de cache compartilhado, determina "organização preferida" via Git remote do workspace
- [src/extension/agents/vscode-node/promptFileContrib.ts](src/extension/agents/vscode-node/promptFileContrib.ts) — Ponto de registro para todos os providers
- [src/extension/vscode.proposed.chatPromptFiles.d.ts](src/extension/vscode.proposed.chatPromptFiles.d.ts) — API proposta com `registerCustomAgentProvider`, `registerInstructionsProvider`, `registerSkillProvider`

### Ordem de Precedência (Confirmada no Código)

```
Workspace (.github/agents/)     → MAIOR precedência (vence sempre)
       ↓
User-level (~/.copilot/)        → Precedência média
       ↓
Org-level (.github-private)     → MENOR precedência
```

**Implicação:** Agentes centralizados NÃO substituem agentes de workspace. Se um repo tem seu próprio `planning.agent.md`, ele será usado ao invés do centralizado.

---

## Estrutura Atual do SDD Bootstrap v5.0.1

### Artefatos Instalados por Repo (Atual)

| Tipo | Quantidade | Diretório | Arquivos |
|------|-----------|-----------|----------|
| Agentes | 8 | `.github/agents/` | `planning.agent.md`, `architecture.agent.md`, `implementation.agent.md`, `review.agent.md`, `test.agent.md`, `orchestrator.agent.md`, `bootstrap.agent.md`, `product-owner.agent.md` |
| Skills | 14 | `.github/skills/` | `toon-validation/`, `feature-spec/`, `code-review/`, `batch-planning/`, `test-generation/`, `architecture-decision/`, `code-generation/`, `jira-sync/`, `readme-evolution/`, `confluence-sync/`, `dor-epic-validation/`, `dor-story-validation/`, `dod-validation/`, `doa-metrics/` |
| Prompts | 3+ | `.github/prompts/` | `domain.prompt.md`, `test.prompt.md`, `use-case.prompt.md` (+ contexto-dependentes) |
| Specs | 4 dirs | `.github/specs/` | `features/`, `acceptance/`, `architecture/`, `docs/` |
| Config | 7+ | `.sdd/` | `context.md`, `version.md`, `framework.md`, `language.md`, `business-context.md`, `domain-vocabulary.md`, `technical-context.md`, `definitions/` |
| Makefile | 1 | raiz | `sdd.mk` (2456 linhas) |

### O que CENTRALIZAR vs. O que fica POR REPO

| Centralizar (Compartilhado) | Manter por Repo (Específico) |
|------|------|
| 8 agentes `.agent.md` | `.sdd/context.md` (nome do projeto, linguagem, framework) |
| 14 skills `SKILL.md` | `.sdd/version.md` |
| 3 prompts core `.prompt.md` | `.sdd/domain-vocabulary.md` |
| | `.sdd/business-context.md` |
| | `.sdd/technical-context.md` |
| | `.github/copilot-instructions.md` |
| | `.github/specs/` (specs geradas) |
| | `sdd.mk` (continua necessário para targets de validação) |

---

## Estado Atual do Fork (primoPP/SDD-Comand-Center)

O repositório [primoPP/SDD-Comand-Center](https://github.com/primoPP/SDD-Comand-Center) é um fork de `microsoft/vscode-copilot-chat`.

**Estrutura `.github/` atual do fork:**
- `.github/CODENOTIFY` — Regras de notificação por arquivo
- `.github/CODEOWNERS` — Proprietários de código
- `.github/ISSUE_TEMPLATE/` — Templates de issues
- `.github/commands.json` — Configurações de bot
- `.github/copilot-instructions.md` — Instruções gerais do Copilot (17KB, herdado do microsoft/vscode-copilot-chat)
- `.github/dependabot.yml` — Config do Dependabot
- `.github/instructions/` — Instruções de código
- `.github/prompts/` — Prompts existentes
- `.github/workflows/` — CI/CD

**O que FALTA no fork para centralização:**
- ❌ Não tem `.github/agents/` (nenhum agente SDD instalado)
- ❌ Não tem `.github/skills/` (nenhum skill SDD instalado)
- ❌ Não tem `.sdd/` (sem config SDD)
- ❌ Não tem `sdd.mk` (sem Makefile SDD)
- ❌ Não tem `.claude/` com conteúdo SDD relevante

**Ações necessárias no fork ANTES da implementação:**
1. Instalar SDD Bootstrap v5.0.1 no fork (`make -f sdd.mk install`)
2. Adicionar novos targets de centralização ao `sdd.mk`
3. Criar scripts de instalação centralizada
4. Documentar o processo no README do fork

---

## Regras de Negócio

| ID | Regra | Condição | Resultado Sucesso | Resultado Falha | Código Erro |
|----|-------|----------|-------------------|-----------------|-------------|
| BR-001 | Instalação User-Level | Developer executa `install-central` | Agentes, skills e prompts copiados para `~/.copilot/` e settings do VS Code configurados | Erro com mensagem descritiva | `CENTRAL_INSTALL_FAILED` |
| BR-002 | Configuração VS Code Settings | `install-central` completa a cópia dos arquivos | `chat.instructionsFilesLocations` e `chat.agentSkillsLocations` atualizados no settings.json do usuário | Arquivo de settings não encontrado ou sem permissão | `SETTINGS_UPDATE_FAILED` |
| BR-003 | Push para Org GitHub | Developer executa `install-org` com acesso à org | Agentes são enviados para repo `.github-private` na org, disponíveis para todos os membros | Falha de autenticação ou repo não encontrado | `ORG_PUSH_FAILED` |
| BR-004 | Precedência de Workspace | Agentes centralizados e por-repo coexistem para mesmo nome | Agente do workspace (`.github/agents/`) tem precedência sobre centralizado | N/A (comportamento nativo do VS Code) | N/A |
| BR-005 | Init Leve por Repo | Developer executa `install-project-context` em repo com config centralizada | Apenas `.sdd/` e `copilot-instructions.md` são criados, NENHUM agente/skill/prompt copiado | Falha se `.sdd/` já existe com install completo | `CONTEXT_INIT_FAILED` |
| BR-006 | Status de Centralização | Developer executa `central-status` | Mostra estratégia ativa, lista de arquivos centralizados, estado dos settings VS Code | Nenhuma centralização detectada | `NO_CENTRAL_CONFIG` |

---

## Análise de Batches

### Total de Batches: 3

| ID | Nome | Escopo | Estimativa | Dependências | 🔧 | 🧪 | 🚀 |
|----|------|--------|------------|--------------|----|----|----|
| batch-a | Centralização User-Level | `install-central` + `uninstall-central` + `central-status` + cópia para `~/.copilot/` + config VS Code settings + documentação | 2d | Nenhuma | ✅ | ✅ | ✅ |
| batch-b | Centralização Org-Level | `install-org` + integração GitHub API + push para `.github-private` + template org agent + prompt `@bootstrap org-sync` | 2d | batch-a | ✅ | ✅ | ✅ |
| batch-c | Init Leve por Repo | `install-project-context` + init `.sdd/` mínimo + geração de `copilot-instructions.md` + setup vocabulário domínio | 1d | batch-a | ✅ | ✅ | ✅ |

### Detalhamento dos Batches

#### Batch A: Centralização User-Level
- **Classificação:** Independente
- **Estimativa:** 2 dias
- **Dependências:** Nenhuma
- **Escopo:**
  - Target `install-central` no `sdd.mk` — copia 8 agentes para `~/.copilot/agents/`, 14 skills para `~/.copilot/skills/`, 3 prompts para `~/.copilot/prompts/`
  - Target `uninstall-central` — remove arquivos e reverte settings
  - Target `central-status` — mostra estado atual da centralização
  - Script de atualização do VS Code `settings.json` (adiciona `chat.instructionsFilesLocations` e `chat.agentSkillsLocations`)
  - Suporte ao flag `FORCE=true` para sobrescrever arquivos modificados
  - Idempotência (safe para re-executar)
  - Documentação de uso
- **Validação Three Pillars:**
  - 🔧 Codificável: ✅ Targets de Makefile, scripts shell, documentação
  - 🧪 Testável: ✅ Pode testar se arquivos foram copiados, settings atualizados, agentes descobertos
  - 🚀 Deployável: ✅ Funciona independente, não afeta repos existentes

#### Batch B: Centralização Org-Level
- **Classificação:** Dependente
- **Estimativa:** 2 dias
- **Dependências:** Requer batch-a (reutiliza estrutura de arquivos e lógica de config)
- **Escopo:**
  - Target `install-org` no `sdd.mk` — usa GitHub CLI (`gh`) para push de agentes para `<org>/.github-private`
  - Integração com GitHub API via `gh api` para listar/criar/atualizar arquivos
  - Verificação de existência do repo `.github-private` com mensagem de erro clara se não existir
  - Template de configuração organizacional
  - Prompt `@bootstrap org-sync` para sincronização via chat
- **Validação Three Pillars:**
  - 🔧 Codificável: ✅ Target Makefile + scripts de integração GitHub API
  - 🧪 Testável: ✅ Pode testar com org de teste ou mock do `gh` CLI
  - 🚀 Deployável: ✅ Funciona independente do batch-c, requer apenas batch-a

#### Batch C: Init Leve por Repo
- **Classificação:** Dependente
- **Estimativa:** 1 dia
- **Dependências:** Requer batch-a (centralização deve estar disponível primeiro)
- **Escopo:**
  - Target `install-project-context` no `sdd.mk` — cria APENAS `.sdd/` com arquivos de contexto e `copilot-instructions.md`
  - NÃO copia agentes, skills ou prompts
  - Aceita `PROJECT_NAME` como parâmetro para personalizar `context.md`
  - Gera `copilot-instructions.md` com referências ao SDD mas sem definições de agentes embarcadas
  - Detecta se já existe install completo e avisa o usuário
- **Validação Three Pillars:**
  - 🔧 Codificável: ✅ Target Makefile + templates de contexto
  - 🧪 Testável: ✅ Pode verificar que APENAS .sdd/ e copilot-instructions.md foram criados
  - 🚀 Deployável: ✅ Independente para deploy, requer apenas que batch-a tenha sido executado pelo dev

### Estratégia de Implementação

**Ordem Recomendada:** batch-a → batch-b e batch-c (paralelos)

**Oportunidades de Paralelização:**
- Wave 1: batch-a (fundação — install/uninstall/status user-level)
- Wave 2 (após Wave 1): batch-b e batch-c podem ser paralelizados

**Caminho Crítico:** batch-a → batch-b = 4 dias

**Com Paralelização:** batch-a → (batch-b ‖ batch-c) = 3 dias

---

## Fluxo Principal

### Estratégia 1: User-Level (install-central)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Developer clona primoPP/SDD-Comand-Center                   │
│    - UMA VEZ por máquina                                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Developer executa:                                           │
│    make -f sdd.mk install-central                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. Sistema copia arquivos:                                      │
│    - 8 agentes   → ~/.copilot/agents/                           │
│    - 14 skills   → ~/.copilot/skills/                           │
│    - 3 prompts   → ~/.copilot/prompts/                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. Sistema atualiza VS Code settings.json:                      │
│    - chat.instructionsFilesLocations ← ~/.copilot/              │
│    - chat.agentSkillsLocations ← ~/.copilot/skills/             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. ✅ PRONTO. Developer abre qualquer workspace no VS Code.     │
│    Agentes SDD disponíveis automaticamente via @planning,       │
│    @architecture, @implementation, @review, @test, etc.         │
└─────────────────────────────────────────────────────────────────┘
```

### Estratégia 2: Org-Level (install-org)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Admin da org executa:                                        │
│    make -f sdd.mk install-org ORG_NAME=PicPay                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Sistema verifica acesso ao repo PicPay/.github-private       │
│    via gh auth status                                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴──────────┐
            SUCESSO                 FALHA
                    │                    │
                    ▼                    ▼
┌───────────────────────────────┐  ┌───────────────────────────────┐
│ 3a. Push agentes SDD para    │  │ 3b. Erro ORG_PUSH_FAILED      │
│ PicPay/.github-private/      │  │ "Verify gh auth status"       │
│ .github/agents/*.agent.md    │  │                               │
└───────────────────────────────┘  └───────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. VS Code de TODOS os devs da org detecta agentes             │
│    automaticamente via polling a cada 5 min                     │
│    (GitHubOrgCustomAgentProvider)                               │
└─────────────────────────────────────────────────────────────────┘
```

### Estratégia 3: Init Leve por Repo (install-project-context)

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Developer já tem centralização (user ou org) configurada.    │
│    Quer usar SDD num novo repo sem instalar tudo.               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. Developer executa:                                           │
│    make -f sdd.mk install-project-context                       │
│                     PROJECT_NAME=meu-servico                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. Sistema cria APENAS:                                         │
│    - .sdd/context.md (com PROJECT_NAME)                         │
│    - .sdd/version.md                                            │
│    - .sdd/domain-vocabulary.md (template)                       │
│    - .github/copilot-instructions.md                            │
│    ❌ NÃO copia agentes, skills ou prompts                      │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. ✅ PRONTO. Agentes SDD vem do nível centralizado.            │
│    Contexto do projeto vem do .sdd/ local.                      │
│    Developer pode usar @planning, @architecture etc.            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Endpoints da API

N/A — Esta feature é baseada em CLI (Makefile targets) e não possui endpoints HTTP.

### Comandos Makefile (Equivalente a Endpoints)

#### `make -f sdd.mk install-central`

Instala agentes, skills e prompts SDD no diretório home do usuário.

**Parâmetros:**
```bash
BOOTSTRAP_SOURCE=/path/to/sdd-source  # Opcional, default: diretório atual
FORCE=true                            # Opcional, default: false
```

**Saída (sucesso):**
```
✅ SDD Central Install Complete
   Agents:  8 files → ~/.copilot/agents/
   Skills:  14 dirs → ~/.copilot/skills/
   Prompts: 3 files → ~/.copilot/prompts/
   Settings: VS Code user settings.json updated
```

#### `make -f sdd.mk install-org`

Publica agentes SDD no repositório `.github-private` da organização GitHub.

**Parâmetros:**
```bash
ORG_NAME=PicPay                        # Opcional, default: detect from git remote
REPO_NAME=.github-private              # Opcional, default: .github-private
BOOTSTRAP_SOURCE=/path/to/sdd-source   # Opcional, default: diretório atual
```

**Saída (sucesso):**
```
✅ SDD Org Install Complete
   Org: PicPay
   Repo: .github-private
   Agents pushed: 8 files
   VS Code will discover agents via polling (up to 5 min)
```

#### `make -f sdd.mk install-project-context`

Inicializa contexto leve de projeto sem copiar agentes/skills/prompts.

**Parâmetros:**
```bash
PROJECT_NAME=meu-servico               # Opcional, default: basename do diretório
PROJECT_DIR=.                          # Opcional, default: diretório atual
```

**Saída (sucesso):**
```
✅ SDD Project Context Initialized
   Created: .sdd/context.md
   Created: .sdd/version.md
   Created: .sdd/domain-vocabulary.md
   Created: .github/copilot-instructions.md
   Agents: Using centralized (not copied)
```

#### `make -f sdd.mk uninstall-central`

Remove instalação centralizada e reverte configurações.

**Parâmetros:**
```bash
STRATEGY=user|org|all                  # Obrigatório
```

#### `make -f sdd.mk central-status`

Exibe estado atual da centralização.

**Saída (exemplo):**
```
📊 SDD Centralization Status
   Strategy: user-level
   Agents:   8/8 synced (v5.0.1)
   Skills:   14/14 synced
   Prompts:  3/3 synced
   VS Code Settings:
     chat.instructionsFilesLocations: ~/.copilot/ ✅
     chat.agentSkillsLocations: ~/.copilot/skills/ ✅
   Org-level: not configured
```

---

## Entidades de Domínio

### Estrutura de Diretórios Centralizada

```
~/.copilot/                              # Raiz centralizada (user-level)
├── agents/                              # 8 agentes SDD
│   ├── planning.agent.md
│   ├── architecture.agent.md
│   ├── implementation.agent.md
│   ├── review.agent.md
│   ├── test.agent.md
│   ├── orchestrator.agent.md
│   ├── bootstrap.agent.md
│   └── product-owner.agent.md
├── skills/                              # 14 skills SDD
│   ├── toon-validation/SKILL.md
│   ├── feature-spec/SKILL.md
│   ├── code-review/SKILL.md
│   ├── batch-planning/SKILL.md
│   ├── test-generation/SKILL.md
│   ├── architecture-decision/SKILL.md
│   ├── code-generation/SKILL.md
│   ├── jira-sync/SKILL.md
│   ├── readme-evolution/SKILL.md
│   ├── confluence-sync/SKILL.md
│   ├── dor-epic-validation/SKILL.md
│   ├── dor-story-validation/SKILL.md
│   ├── dod-validation/SKILL.md
│   └── doa-metrics/SKILL.md
└── prompts/                             # 3 prompts core
    ├── domain.prompt.md
    ├── test.prompt.md
    └── use-case.prompt.md
```

### Estrutura Leve por Repo (install-project-context)

```
meu-servico/                             # Repo do projeto
├── .sdd/                               # Apenas contexto
│   ├── context.md                       # Nome, linguagem, framework
│   ├── version.md                       # Versão SDD
│   └── domain-vocabulary.md             # Vocabulário específico do projeto
├── .github/
│   ├── copilot-instructions.md          # Instruções Copilot do projeto
│   └── specs/                           # Specs geradas pelo SDD
│       ├── features/
│       ├── acceptance/
│       ├── architecture/
│       └── docs/
└── sdd.mk                              # Makefile (para validate e outros targets)
```

### Enumeração de Estratégias

| Estratégia | Descrição |
|------------|-----------|
| `user-level` | Agentes instalados em `~/.copilot/`, configurados via VS Code settings |
| `org-level` | Agentes publicados em `<org>/.github-private`, detectados via polling |
| `hybrid` | Combinação de user-level + org-level |
| `none` | Nenhuma centralização (modo atual — install completo por repo) |

---

## Cenários de Erro

| Código | HTTP Status | Mensagem | Gatilho |
|--------|-------------|----------|---------|
| `CENTRAL_INSTALL_FAILED` | N/A | "Error: SDD source not found at [path]. Clone primoPP/SDD-Comand-Center first." | Diretório fonte não encontrado ou sem agentes |
| `SETTINGS_UPDATE_FAILED` | N/A | "Error: Cannot update VS Code settings at [path]. Check file permissions." | settings.json read-only ou malformado |
| `ORG_PUSH_FAILED` | N/A | "Error: Cannot push to [org]/.github-private. Verify gh auth status and org write access." | Falha de autenticação GitHub API |
| `CONTEXT_INIT_FAILED` | N/A | "Error: Cannot create .sdd/ directory. Check permissions and disk space." | Erro de permissão ou disco cheio |
| `NO_CENTRAL_CONFIG` | N/A | "No SDD centralization detected. Run 'make -f sdd.mk install-central' first." | Nenhuma centralização configurada |

---

## Edge Cases

| Caso | Comportamento Esperado |
|------|------------------------|
| `~/.copilot/` não existe antes do `install-central` | Diretório criado automaticamente com permissões corretas |
| VS Code `settings.json` não existe ou está vazio | Arquivo criado com JSON mínimo válido contendo os settings de centralização |
| Repo `.github-private` não existe na org | `install-org` falha com erro claro sugerindo criação do repo |
| `.sdd/` já existe com install completo quando roda `install-project-context` | Comando detecta install existente e avisa o usuário com opção de converter para modo leve |
| Developer sem internet durante `install-central` | Comando funciona a partir do clone local sem necessidade de rede |
| Agente centralizado tem versão diferente do workspace | Versão do workspace prevalece (comportamento nativo VS Code) |
| Múltiplos VS Code windows abertos durante install | Cada window precisa de reload para detectar novos settings |

---

## Premissas

1. VS Code v1.108+ suporta `chat.instructionsFilesLocations` e `chat.agentSkillsLocations` settings para descoberta de arquivos externos
2. Repositório `<org>/.github-private` suporta hospedagem de agentes customizados via `GitHubOrgCustomAgentProvider` existente no VS Code
3. VS Code faz polling de agentes organizacionais a cada 5 minutos via mecanismo existente
4. Diretório home `~/.copilot/` é gravável e acessível no sistema do desenvolvedor
5. VS Code `settings.json` está na localização padrão por OS (`~/Library/Application Support/Code/User/settings.json` no macOS)
6. Repositório fonte de centralização é `primoPP/SDD-Comand-Center` (fork pessoal de microsoft/vscode-copilot-chat) que hospedará os agentes e skills SDD canônicos
7. GitHub CLI (`gh`) está instalado e autenticado para uso do `install-org`

---

## Eventos de Domínio

| Evento | Gatilho | Payload |
|--------|---------|---------|
| `CentralInstallCompleted` | `install-central` ou `install-org` finaliza com sucesso | `strategy:string, fileCount:int, settingsUpdated:bool` |
| `CentralUninstallCompleted` | `uninstall-central` finaliza com sucesso | `strategy:string, filesRemoved:int, settingsReverted:bool` |

---

## Referência Rápida de Código-Fonte (VS Code Copilot Chat)

Para implementação futura, estes são os arquivos-chave na codebase do vscode-copilot-chat que lidam com descoberta de agentes:

| Arquivo | Responsabilidade |
|---------|------------------|
| `src/platform/customInstructions/common/customInstructionsService.ts` | Serviço core de descoberta de instruções/agentes — processa `chat.instructionsFilesLocations` |
| `src/platform/customInstructions/common/promptTypes.ts` | Constantes de paths: `PERSONAL_SKILL_FOLDERS = ['.copilot/skills', '.claude/skills']` |
| `src/extension/agents/vscode-node/githubOrgCustomAgentProvider.ts` | Polling de agentes na org GitHub, usa `octoKitService.getCustomAgents()` |
| `src/extension/agents/vscode-node/githubOrgInstructionsProvider.ts` | Polling de instruções organizacionais (intervalo 2 min) |
| `src/extension/agents/vscode-node/githubOrgChatResourcesService.ts` | Cache compartilhado, determina org preferida via Git remote |
| `src/extension/agents/vscode-node/promptFileContrib.ts` | Ponto de registro para todos os providers de agentes/skills/instructions |
| `src/extension/vscode.proposed.chatPromptFiles.d.ts` | API proposta: `registerCustomAgentProvider`, `registerInstructionsProvider`, `registerSkillProvider` |

---

## Perguntas em Aberto

⚠️ **SEÇÃO OBRIGATÓRIA** — Todas as lacunas identificadas na Gap Analysis.

**MATH CHECK:**
- ❓ no scan: 6
- Linhas nesta tabela: 6
- MATCH: ✅

| # | Scan-ID | Pergunta | Bloqueante? | Categoria | Impacto |
|---|---------|----------|-------------|-----------|---------|
| 1 | A-BC-03 | Qual é o passo-a-passo exato do primeiro uso em um novo repo após instalação centralizada? | 🟡 Não | Contexto de Negócio | Pode ser inferido do fluxo de install existente |
| 2 | B-CDS-02 | Comandos de install devem ser idempotentes (safe para re-executar) ou falhar se já instalado? | 🟡 Não | Detalhe Técnico | Default: idempotente (padrão da indústria) |
| 3 | B-CDS-04 | Como deve funcionar a migração de install completo por-repo para centralizado em repos existentes? | 🟡 Não | Detalhe Técnico | Pode documentar guia de migração separadamente |
| 4 | B-INT-01 | Quais endpoints específicos da GitHub API são necessários para push de agentes para `.github-private`? | 🟡 Não | Detalhe Técnico | Pode usar padrões octokit existentes na codebase |
| 5 | B-INT-03 | O `install-org` deve suportar modo dry-run para preview do que será enviado? | 🟡 Não | Detalhe Técnico | Nice-to-have que pode ser adicionado depois |
| 6 | A-EC-02 | Se a API do GitHub org não estiver disponível durante `install-org`, deve fazer retry ou falhar imediatamente? | 🟡 Não | Detalhe Técnico | Default: falhar com mensagem clara (padrão) |
