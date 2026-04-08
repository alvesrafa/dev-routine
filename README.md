# dev-routine

> 🇧🇷 [Português](#português) | 🇺🇸 [English](#english)

---

<a name="português"></a>

# 🇧🇷 Português

Skills de desenvolvimento para times que usam Claude Code — padronização de fluxos de planejamento, revisão, debugging e auditoria.

## Skills disponíveis

| Skill         | Comando                    | Descrição                                                                      | Arquivos de contexto usados                                          |
| ------------- | -------------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| `plan-task`   | `/plan-task "descrição"`   | Gera plano detalhado com checklist, arquivos afetados, edge cases e estimativa | `project.md`, `conventions.md`, `architecture.md`, `known-issues.md` |
| `code-review` | `/code-review "descrição"` | Valida o fluxo descrito, revisa o código e emite veredito de PR (✅ / ⚠️ / ❌) | `conventions.md`, `architecture.md`, `known-issues.md`               |
| `fix-bug`     | `/fix-bug "descrição"`     | Consulta known-issues, gera hipóteses ordenadas e plano de correção            | `known-issues.md`, `architecture.md`, `project.md`                   |
| `audit-code`  | `/audit-code "descrição"`  | Checklist de segurança, banco, performance, filas e K8s antes do deploy        | `project.md`, `conventions.md`, `architecture.md`, `known-issues.md` |

A descrição é **opcional** em todas as skills, mas melhora consideravelmente a qualidade da resposta.

---

## Instalação

**Pré-requisito:** Node.js instalado.

### 1. Instalar globalmente (uma vez por máquina)

```bash
npx skills add alvesrafa/dev-routine --all -g -a claude-code -y
```

As skills ficam em `~/.claude/skills/` e ficam disponíveis em todos os projetos.

### 2. Verificar instalação

```bash
npx skills list -g -a claude-code
```

### 3. Atualizar

```bash
npx skills add alvesrafa/dev-routine --all -g -a claude-code -y
```

O comando sobrescreve a versão anterior automaticamente.

---

## Configuração por projeto

As skills leem arquivos de contexto dentro da pasta `.claude/` na raiz de cada projeto para especializar as respostas ao stack e às decisões técnicas específicas daquele projeto.

### Estrutura esperada

```
seu-projeto/
├── .claude/
│   ├── project.md
│   ├── conventions.md
│   ├── architecture.md
│   └── known-issues.md
└── ... (resto do projeto)
```

### Setup manual

Crie a pasta `.claude/` na raiz do seu projeto e adicione os arquivos `project.md`, `conventions.md`, `architecture.md` e `known-issues.md`. Use a estrutura esperada acima como guia.

### Arquivos de contexto

| Arquivo           | Obrigatório | Quem preenche       | Skills que usam                           | O que documentar                                                                                |
| ----------------- | ----------- | ------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `project.md`      | ✅ Sim      | Tech lead           | Todas                                     | Stack, ambientes, repositórios, módulos de negócio, integrações externas                        |
| `conventions.md`  | ✅ Sim      | Time                | `/code-review`, `/plan-task`              | Padrões de código, nomenclatura, regras de PR, antipadrões proibidos                            |
| `architecture.md` | ✅ Sim      | Tech lead           | `/plan-task`, `/audit-code`               | Padrões arquiteturais, fluxos dos módulos, infraestrutura K8s, decisões técnicas e seus motivos |
| `known-issues.md` | ⚡ Auto     | Skills (automático) | `/fix-bug`, `/code-review`, `/audit-code` | Problemas já encontrados e suas soluções — populado automaticamente, não editar manualmente     |

> **`known-issues.md`** começa vazio. As skills `/code-review`, `/fix-bug` e `/audit-code` adicionam entradas automaticamente quando detectam problemas com potencial de recorrência. Com o tempo, vira a memória técnica do projeto. Commite junto com o código.

---

## Uso no dia a dia

```bash
# Antes de começar uma tarefa
/plan-task "implementar webhook de notificação para atualizar status do pedido"

# Antes de abrir PR
/code-review "adicionei endpoint POST /orders/:id/cancel com verificação de status"

# Ao encontrar um bug
/fix-bug "job de importação falha silenciosamente para arquivos acima de 50MB"

# Antes de subir para produção
/audit-code "módulo de pagamentos — primeiro deploy em produção"
```

---

## Estrutura do repositório

```
dev-routine/
├── README.md
├── plan-task/
│   └── SKILL.md
├── code-review/
│   └── SKILL.md
├── fix-bug/
│   └── SKILL.md
├── audit-code/
│   └── SKILL.md
└── project-template/
    └── .claude/
        ├── project.md
        ├── conventions.md
        ├── architecture.md
        └── known-issues.md
```

---

---

<a name="english"></a>

# 🇺🇸 English

Development skills for teams using Claude Code — standardized flows for planning, review, debugging, and auditing.

## Available skills

| Skill         | Command                      | Description                                                                             | Context files used                                                   |
| ------------- | ---------------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `plan-task`   | `/plan-task "description"`   | Generates a detailed plan with checklist, affected files, edge cases, and time estimate | `project.md`, `conventions.md`, `architecture.md`, `known-issues.md` |
| `code-review` | `/code-review "description"` | Validates the described flow, reviews code, and issues a PR verdict (✅ / ⚠️ / ❌)      | `conventions.md`, `architecture.md`, `known-issues.md`               |
| `fix-bug`     | `/fix-bug "description"`     | Checks known-issues first, then generates ordered hypotheses and a fix plan             | `known-issues.md`, `architecture.md`, `project.md`                   |
| `audit-code`  | `/audit-code "description"`  | Security, database, performance, queue, and K8s checklist before deploying              | `project.md`, `conventions.md`, `architecture.md`, `known-issues.md` |

The description is **optional** for all skills, but significantly improves response quality.

---

## Installation

**Prerequisite:** Node.js installed.

### 1. Install globally (once per machine)

```bash
npx skills add alvesrafa/dev-routine --all -g -a claude-code -y
```

Skills are installed to `~/.claude/skills/` and become available across all projects.

### 2. Verify installation

```bash
npx skills list -g -a claude-code
```

### 3. Update

```bash
npx skills add alvesrafa/dev-routine --all -g -a claude-code -y
```

This automatically overwrites the previous version.

---

## Per-project setup

Skills read context files from a `.claude/` folder at the root of each project to tailor responses to that project's specific stack and technical decisions.

### Expected structure

```
your-project/
├── .claude/
│   ├── project.md
│   ├── conventions.md
│   ├── architecture.md
│   └── known-issues.md
└── ... (rest of project)
```

### Manual setup

Create the `.claude/` folder at your project root and add the files `project.md`, `conventions.md`, `architecture.md`, and `known-issues.md`. Use the expected structure above as a guide.

### Context files

| File              | Required | Who fills it       | Skills that use it                        | What to document                                                                                  |
| ----------------- | -------- | ------------------ | ----------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `project.md`      | ✅ Yes   | Tech lead          | All                                       | Stack, environments, repositories, business modules, external integrations                        |
| `conventions.md`  | ✅ Yes   | Team               | `/code-review`, `/plan-task`              | Coding standards, naming conventions, PR rules, forbidden antipatterns                            |
| `architecture.md` | ✅ Yes   | Tech lead          | `/plan-task`, `/audit-code`               | Architectural patterns, module flows, K8s infrastructure, technical decisions and their rationale |
| `known-issues.md` | ⚡ Auto  | Skills (automatic) | `/fix-bug`, `/code-review`, `/audit-code` | Known problems and their solutions — auto-populated by skills, do not edit manually               |

> **`known-issues.md`** starts empty. The `/code-review`, `/fix-bug`, and `/audit-code` skills automatically add entries when they detect recurring problems. Over time it becomes the project's technical memory. Commit it alongside your code.

---

## Daily usage

```bash
# Before starting a task
/plan-task "implement webhook notification to update order status"

# Before opening a PR
/code-review "added POST /orders/:id/cancel endpoint with status validation"

# When investigating a bug
/fix-bug "import job silently fails for files over 50MB"

# Before deploying to production
/audit-code "payments module — first production deploy"
```

---

## Repository structure

```
dev-routine/
├── README.md
├── plan-task/
│   └── SKILL.md
├── code-review/
│   └── SKILL.md
├── fix-bug/
│   └── SKILL.md
├── audit-code/
│   └── SKILL.md
└── project-template/
    └── .claude/
        ├── project.md
        ├── conventions.md
        ├── architecture.md
        └── known-issues.md
```
