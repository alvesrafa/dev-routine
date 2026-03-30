---
name: fix-bug
description: Guia a investigação e correção de bugs de forma estruturada. Use esta skill quando o usuário acionar /fix-bug com uma descrição, ou quando reportar um erro, comportamento inesperado, ou pedir para debugar algo. Sempre usar quando a mensagem começar com /fix-bug.
---

# Skill: /fix-bug

## Acionamento

O usuário aciona com:

```
/fix-bug "descrição do comportamento inesperado"
```

A descrição é opcional mas crítica para qualidade da investigação. Sem ela, peça antes de continuar.

## Leitura de contexto do projeto

Antes de qualquer análise, procure e leia os seguintes arquivos **se existirem**:

- `.claude/project.md` — stack, ambientes, módulos
- `.claude/known-issues.md` — **leitura prioritária**: verificar se o bug já foi visto antes
- `.claude/architecture.md` — para entender fluxos e dependências
- `.claude/conventions.md` — para sugerir correção dentro dos padrões

## Fase 0: Verificação em known-issues

**Sempre** verifique primeiro o `.claude/known-issues.md`.

Se o bug bater com algo registrado:

- Informe que o problema já foi documentado
- Mostre a solução registrada
- Pergunte se a solução anterior não resolveu ou se o contexto mudou

Se não encontrar correspondência, prossiga com a investigação.

## Fase 1: Entendimento do bug

Com base na descrição fornecida, estruture:

**Comportamento esperado:** o que deveria acontecer  
**Comportamento atual:** o que está acontecendo  
**Ambiente:** onde foi observado (local, HML, prod — inferir do contexto se não informado)  
**Frequência:** sempre, às vezes, em condição específica (inferir se possível)

Se algum ponto for ambíguo, faça **no máximo 2 perguntas** antes de prosseguir.

## Fase 2: Hipóteses de causa

Liste as hipóteses ordenadas da **mais provável para menos provável**, considerando:

- O stack do projeto (`.claude/project.md`)
- Padrões arquiteturais conhecidos (`.claude/architecture.md`)
- Problemas já documentados em `.claude/known-issues.md`
- A descrição do comportamento

Para cada hipótese:

```
### Hipótese N — [título curto]
Probabilidade: Alta / Média / Baixa
Explicação: por que isso poderia causar o comportamento descrito
Como verificar: comando, log, trecho de código para inspecionar
```

## Fase 3: Plano de diagnóstico

Passos concretos para confirmar ou descartar cada hipótese, em ordem de custo/complexidade crescente:

1. Começar pelo que pode ser verificado sem código (logs, banco, fila)
2. Depois verificar código e fluxo
3. Por último: reproduzir em ambiente controlado

Preferir comandos que possam ser rodados **no terminal do pod** quando se tratar de ambiente K8s, evitando commits desnecessários.

## Fase 4: Plano de correção

Após diagnóstico (ou com hipótese de alta confiança):

- **Onde corrigir:** arquivo(s) e função(ões) específicas
- **O que mudar:** descrição da mudança necessária
- **Impacto colateral:** o que mais pode ser afetado pela correção
- **Como testar:** passos para confirmar que o bug foi resolvido

## Fase 5: Registro em known-issues

Se o bug for novo e tiver potencial de se repetir, adicione automaticamente em `.claude/known-issues.md`:

```markdown
## [Categoria — título curto]

Descrição: comportamento observado e causa raiz.
Contexto: onde e quando tende a aparecer.
Solução: como corrigir.
Detectado em: [fix-bug]
```

Informe o usuário: _"⚠️ Registrado em known-issues: [título]"_

Se o arquivo não existir e houver algo para registrar, crie-o.

## Formato de saída

```
## Known Issues
[Problema já documentado / Não encontrado — prosseguindo]

## Entendimento do Bug
Esperado: ...
Atual: ...
Ambiente: ...

## Hipóteses
[lista ordenada por probabilidade]

## Plano de Diagnóstico
[passos numerados]

## Plano de Correção
[após diagnóstico ou com alta confiança na hipótese]

## Known Issues
[novo registro adicionado, ou "Nenhum registro necessário"]
```
