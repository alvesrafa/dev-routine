---
name: code-review
description: Realiza code review completo de implementações. Use esta skill quando o usuário acionar /code-review, ou quando pedir para revisar código, verificar uma implementação, ou analisar um PR. Sempre usar quando a mensagem começar com /code-review.
---

# Skill: /code-review

## Acionamento

O usuário aciona com:

```
/code-review "descrição do que foi implementado"
```

A descrição é opcional mas permite validar se o fluxo descrito faz sentido antes mesmo de analisar o código.

## Leitura de contexto do projeto

Antes de qualquer análise, procure e leia os seguintes arquivos **se existirem**:

- `.claude/project.md` — stack, ambientes, módulos
- `.claude/conventions.md` — padrões obrigatórios do time
- `.claude/architecture.md` — padrões arquiteturais esperados
- `.claude/known-issues.md` — problemas conhecidos (para identificar recorrências)

## Fase 1: Validação do fluxo descrito (se descrição fornecida)

Se o usuário passou uma descrição, **antes de analisar o código**:

1. Entenda o fluxo descrito em palavras
2. Avalie se o fluxo faz sentido técnico para o objetivo declarado
3. Identifique gaps lógicos, casos não cobertos, ou abordagem questionável
4. Emita um parecer: ✅ Fluxo coerente / ⚠️ Fluxo com ressalvas / ❌ Fluxo problemático

Não reprove o fluxo por preferência estética — apenas por problemas funcionais ou de segurança reais.

## Fase 2: Análise do código

Analise o código fornecido (diff, arquivos colados, ou contexto da conversa) nas seguintes dimensões:

### 🔴 Crítico — Bloqueia o PR

Problemas que **não devem ir para produção**:

- Vulnerabilidades de segurança (SQL injection, dados expostos, auth bypassável)
- Perda de dados (delete sem soft delete quando esperado, truncate acidental)
- Race conditions em jobs/filas
- Migrations sem `down()` ou que podem causar lock em tabela grande
- Secrets ou credenciais hardcoded
- Quebra de contrato de API sem versionamento
- Lógica que contradiz diretamente o requisito descrito

### 🟡 Importante — Deve ser corrigido antes do merge

- Violação das convenções do time (`.claude/conventions.md`)
- Ausência de tratamento de erro em pontos críticos
- Query N+1 ou operação pesada dentro de loop
- Job/comando sem idempotência quando deveria ter
- Falta de validação de input em endpoint público
- Código que vai contra padrões arquiteturais do projeto

### 🟢 Sugestão — Melhoria opcional

- Oportunidade de reuso
- Nomenclatura confusa mas funcional
- Comentário desnecessário ou ausente onde faria diferença
- Simplificação possível sem risco

## Fase 3: Detecção de problemas recorrentes

Após a análise, verifique se algum problema encontrado:

- Já aparece no `.claude/known-issues.md` do projeto (indicar que é recorrente)
- É **novo e tem potencial de se repetir** em outros contextos do projeto

### Atualização automática do known-issues

Se identificar um problema novo e recorrente que **ainda não está** em `.claude/known-issues.md`, adicione automaticamente uma entrada no arquivo com o formato:

```markdown
## [Categoria do problema]

Descrição objetiva do problema encontrado.
Contexto: onde tende a aparecer no projeto.
Solução: como resolver ou prevenir.
Detectado em: [data aproximada ou "code-review"]
```

Informe o usuário sobre a adição com: _"⚠️ Adicionado em known-issues: [título do problema]"_

Se `.claude/known-issues.md` não existir e houver algo para registrar, crie o arquivo.

## Fase 4: Parecer final

Emita um dos três vereditos:

**✅ APROVADO** — Pode seguir para PR sem ressalvas críticas  
**⚠️ APROVADO COM RESSALVAS** — Pode abrir PR mas os itens amarelos devem ser resolvidos  
**❌ BLOQUEADO** — Contém itens críticos que precisam ser resolvidos antes do PR

Liste os itens que motivaram o veredito de forma numerada e acionável.

## Formato de saída

```
## Validação do Fluxo (se descrição fornecida)
[parecer do fluxo]

## Itens Críticos 🔴
[lista ou "Nenhum"]

## Itens Importantes 🟡
[lista ou "Nenhum"]

## Sugestões 🟢
[lista ou "Nenhum"]

## Known Issues
[novos itens adicionados, ou "Nenhum novo problema registrado"]

## Veredito
[✅ / ⚠️ / ❌] + justificativa em 1-2 linhas
```
