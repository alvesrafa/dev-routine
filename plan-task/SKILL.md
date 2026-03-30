---
name: plan-task
description: Planeja tarefas de desenvolvimento de forma estruturada para times. Use esta skill quando o usuário acionar /plan-task com uma descrição, ou quando pedir para planejar, detalhar, ou quebrar uma tarefa de desenvolvimento em etapas. Sempre usar quando a mensagem começar com /plan-task.
---

# Skill: /plan-task

## Acionamento

O usuário aciona com:

```
/plan-task "descrição da tarefa"
```

A descrição é opcional mas melhora muito a qualidade do plano.

## Leitura de contexto do projeto

Antes de qualquer coisa, procure e leia os seguintes arquivos **se existirem** no projeto atual:

- `.claude/project.md` — stack, ambientes, módulos principais
- `.claude/conventions.md` — padrões de código do time
- `.claude/architecture.md` — decisões técnicas e estrutura
- `.claude/known-issues.md` — armadilhas já conhecidas

Se os arquivos não existirem, informe o usuário e continue com contexto genérico.

## O que produzir

### 1. Entendimento da tarefa

Reformule a tarefa com suas próprias palavras para confirmar o entendimento. Se a descrição for ambígua, liste as premissas assumidas.

### 2. Arquivos e módulos envolvidos

Liste os arquivos/pastas prováveis que serão criados ou modificados, com uma linha explicando o motivo de cada um.

### 3. Dependências e pré-requisitos

O que precisa existir ou estar funcionando antes de começar. Inclua: migrations pendentes, serviços externos, permissões, feature flags.

### 4. Checklist de implementação

Passos ordenados e acionáveis. Cada item deve ser pequeno o suficiente para ser feito e testado isoladamente. Use este formato:

```
[ ] Passo 1 — descrição clara
[ ] Passo 2 — descrição clara
...
```

### 5. Edge cases e riscos

Situações que podem quebrar a implementação se não forem consideradas. Priorize por impacto.

### 6. Critérios de aceitação

Como saber que a tarefa está pronta. Deve ser verificável (não subjetivo).

### 7. Estimativa

Faixas de tempo: otimista / realista / pessimista. Justifique se a tarefa for maior que 1 dia.

## Comportamento com descrição fornecida

Se o usuário passou uma descrição, use-a para:

- Identificar o tipo de tarefa (feature, bug, refactor, infra)
- Ajustar o nível de detalhe (bug = mais diagnóstico; feature = mais arquitetura)
- Detectar keywords que indiquem módulos específicos do projeto (ex: "importação", "prestador", "pré-inscrição") e cruzar com `.claude/architecture.md`

## Comportamento sem descrição

Peça ao usuário uma descrição mínima antes de continuar. Não gere um plano genérico vazio.

## Tom e formato

- Direto, sem introduções desnecessárias
- Use markdown com seções numeradas
- Listas de checklist com `[ ]`
- Destaque em **negrito** apenas o que for crítico
