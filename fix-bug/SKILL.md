---
name: fix-bug
description: Guia a investigação e correção de bugs de forma estruturada. Use esta skill quando o usuário acionar /fix-bug com uma descrição, ou quando reportar um erro, comportamento inesperado, ou pedir para debugar algo. Sempre usar quando a mensagem começar com /fix-bug.
---

# Skill: /fix-bug

## Acionamento

```
/fix-bug "descrição do comportamento inesperado"
```

A descrição é crítica para qualidade da investigação. Sem ela, peça antes de continuar.

## Diretriz de resposta

Seja completo mas sucinto. Omita explicações óbvias, evite repetição, vá direto ao ponto. Economize tokens sem perder precisão técnica.

## Leitura de contexto do projeto

Leia **se existirem**:

- `.claude/project.md` — stack, ambientes, módulos
- `.claude/known-issues.md` — **leitura prioritária**
- `.claude/architecture.md` — fluxos e dependências
- `.claude/conventions.md` — padrões do time

## Fase 0: Verificação em known-issues

Verifique `.claude/known-issues.md` primeiro.

- Se bater com algo registrado: mostre a solução e pergunte se o contexto mudou
- Se não: prossiga

## Fase 1: Entendimento do bug

**Esperado:** o que deveria acontecer  
**Atual:** o que está acontecendo  
**Ambiente:** inferir do contexto (local / HML / prod)  
**Frequência:** sempre / às vezes / condição específica

Máximo 2 perguntas se algo for ambíguo.

## Fase 2: Hipóteses de causa

Liste ordenadas da mais para menos provável:

```
### Hipótese N — [título]
Probabilidade: Alta / Média / Baixa
Causa: por que isso explicaria o comportamento
Verificar: comando ou trecho para confirmar
```

## Fase 3: Plano de diagnóstico

Passos concretos em ordem crescente de custo:

1. Logs, banco, fila (sem código)
2. Código e fluxo
3. Reprodução em ambiente controlado

> Preferir comandos executáveis **no terminal do pod** para ambientes K8s.

## Fase 4: Plano de correção

- **Onde:** arquivo(s) e função(ões)
- **O que:** mudança necessária
- **Colateral:** o que mais pode ser afetado
- **Teste:** como confirmar a correção

## Fase 5: Registro em known-issues

Se bug novo com potencial recorrente, adicione em `.claude/known-issues.md`:

```markdown
## [Categoria — título]

Descrição: comportamento e causa raiz.
Contexto: onde tende a aparecer.
Solução: como corrigir.
Detectado em: [fix-bug]
```

Informe: _"⚠️ Registrado em known-issues: [título]"_

## Formato de saída

```
## Known Issues
[Encontrado com solução / Não encontrado]

## Entendimento do Bug
Esperado: ...
Atual: ...
Ambiente: ...

## Hipóteses
[lista ordenada]

## Plano de Diagnóstico
[passos numerados]

## Plano de Correção
[após diagnóstico ou alta confiança na hipótese]

## Known Issues
[novo registro / "Nenhum registro necessário"]
```
