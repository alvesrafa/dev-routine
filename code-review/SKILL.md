---
name: code-review
description: Realiza code review completo de implementações. Use esta skill quando o usuário acionar /code-review, ou quando pedir para revisar código, verificar uma implementação, ou analisar um PR. Sempre usar quando a mensagem começar com /code-review.
---

# Skill: /code-review

## Acionamento

```
/code-review "descrição do que foi implementado"
```

A descrição é opcional mas permite validar o fluxo antes de analisar o código.

## Diretriz de resposta

Seja completo mas sucinto. Omita explicações óbvias, vá direto ao ponto. Economize tokens sem perder precisão técnica. Prefira bullets curtos a parágrafos.

## Leitura de contexto do projeto

Leia **se existirem**:

- `.claude/project.md` — stack, ambientes, módulos
- `.claude/conventions.md` — padrões obrigatórios do time
- `.claude/architecture.md` — padrões arquiteturais esperados
- `.claude/known-issues.md` — para identificar recorrências

## Fase 1: Validação do fluxo descrito (se descrição fornecida)

Antes de analisar o código:

1. Entenda o fluxo descrito
2. Identifique gaps lógicos, casos não cobertos, abordagem questionável
3. Emita parecer: ✅ Coerente / ⚠️ Com ressalvas / ❌ Problemático

Não reprove por preferência estética — apenas por problemas funcionais ou de segurança reais.

## Fase 2: Análise do código

### 🔴 Crítico — Bloqueia o PR

- Vulnerabilidades de segurança (SQL injection, dados expostos, auth bypassável)
- Perda de dados (delete sem soft delete quando esperado, truncate acidental)
- Race conditions em jobs/filas
- Migrations sem `down()` ou que causam lock em tabela grande
- Secrets ou credenciais hardcoded
- Quebra de contrato de API sem versionamento
- Lógica que contradiz diretamente o requisito descrito

### 🟡 Importante — Deve ser corrigido antes do merge

- Violação das convenções do time (`.claude/conventions.md`)
- Ausência de tratamento de erro em pontos críticos
- Query N+1 ou operação pesada dentro de loop
- Job/comando sem idempotência quando deveria ter
- Falta de validação de input em endpoint público
- Violação de padrões arquiteturais do projeto

### 🟢 Sugestão — Melhoria opcional

- Oportunidade de reuso
- Nomenclatura confusa mas funcional
- Simplificação possível sem risco

## Fase 3: Detecção de problemas recorrentes

Verifique se algum problema encontrado já está em `.claude/known-issues.md` ou é novo com potencial recorrente.

### Atualização automática do known-issues

Se novo e recorrente, adicione em `.claude/known-issues.md`:

```markdown
## [Categoria do problema]

Descrição: problema encontrado.
Contexto: onde tende a aparecer.
Solução: como resolver ou prevenir.
Detectado em: [code-review]
```

Informe: _"⚠️ Adicionado em known-issues: [título]"_

Se `.claude/known-issues.md` não existir e houver algo para registrar, crie o arquivo.

## Fase 4: Parecer final

**✅ APROVADO** — Sem críticos  
**⚠️ APROVADO COM RESSALVAS** — Pode abrir PR, itens amarelos devem ser resolvidos  
**❌ BLOQUEADO** — Itens críticos precisam ser resolvidos antes do PR

## Formato de saída

```
## Validação do Fluxo
[parecer / "Sem descrição fornecida"]

## Itens Críticos 🔴
[lista ou "Nenhum"]

## Itens Importantes 🟡
[lista ou "Nenhum"]

## Sugestões 🟢
[lista ou "Nenhum"]

## Known Issues
[novos itens adicionados / "Nenhum novo problema registrado"]

## Veredito
[✅ / ⚠️ / ❌] — justificativa em 1 linha
```
