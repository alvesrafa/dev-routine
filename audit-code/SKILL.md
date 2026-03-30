---
name: audit-code
description: Realiza auditoria completa de segurança, qualidade e infraestrutura antes de subir para produção. Use esta skill quando o usuário acionar /audit-code, ou quando pedir para auditar, verificar se algo está pronto para produção, ou fazer um checklist de qualidade. Sempre usar quando a mensagem começar com /audit-code.
---

# Skill: /audit-code

## Acionamento

O usuário aciona com:

```
/audit-code "descrição do que será auditado"
```

A descrição é opcional. Sem ela, a auditoria é feita sobre o contexto geral disponível (arquivos abertos, diff recente, ou conversa).

## Leitura de contexto do projeto

Antes de qualquer análise, procure e leia os seguintes arquivos **se existirem**:

- `.claude/project.md` — stack, ambientes, módulos
- `.claude/conventions.md` — padrões do time para verificar conformidade
- `.claude/architecture.md` — padrões esperados de infraestrutura e código
- `.claude/known-issues.md` — verificar se algum problema documentado está presente

## Escopo da auditoria

Execute todos os blocos abaixo. Para cada item, marque:

- ✅ OK
- ⚠️ Atenção (não bloqueia, mas deve ser revisado)
- ❌ Crítico (bloqueia deploy)
- ➖ Não aplicável

---

## Bloco 1: Segurança

- [ ] Nenhuma credencial, secret ou API key hardcoded no código
- [ ] Variáveis de ambiente sensíveis lidas de `.env` / secrets do K8s, nunca commitadas
- [ ] Endpoints públicos com validação de input (Form Request / middleware)
- [ ] Autenticação e autorização verificadas nas rotas novas ou modificadas
- [ ] Uploads de arquivo com validação de tipo e tamanho
- [ ] Queries construídas com bindings (nunca interpolação direta de input do usuário)
- [ ] CORS configurado corretamente para o ambiente

## Bloco 2: Banco de dados

- [ ] Toda migration tem `down()` funcional
- [ ] Migrations que alteram tabelas grandes têm estratégia para evitar lock (ex: `ADD COLUMN` com default null)
- [ ] Novos índices criados para campos usados em `WHERE`, `JOIN` ou `ORDER BY` frequentes
- [ ] Soft deletes implementados onde dados não devem ser perdidos permanentemente
- [ ] Nenhum `DB::statement` com DDL sem proteção de transação ou verificação de ambiente
- [ ] Seeds e factories não rodam em produção (guard por `App::environment`)

## Bloco 3: Performance

- [ ] Ausência de N+1 queries (relações eager-loaded onde necessário)
- [ ] Operações pesadas fora do ciclo de request (jobs, commands)
- [ ] Jobs com chunk adequado para volumes grandes
- [ ] Cache utilizado onde dados são lidos frequentemente e mudam pouco
- [ ] Paginação em listagens que podem retornar muitos registros

## Bloco 4: Filas e Jobs

- [ ] Jobs implementam retry com backoff adequado
- [ ] Jobs são idempotentes (reprocessar não gera efeito colateral)
- [ ] Falhas de job logadas com contexto suficiente para debug
- [ ] Fila correta configurada (não tudo na `default`)
- [ ] Timeout de job definido explicitamente quando a operação pode ser longa

## Bloco 5: Infraestrutura e K8s

- [ ] Variáveis de ambiente necessárias documentadas (`.env.example` atualizado)
- [ ] Novos secrets adicionados ao Key Vault e ao manifesto K8s correspondente
- [ ] Recursos de CPU/memória do pod adequados para a carga esperada
- [ ] Health checks (liveness/readiness) não quebrados pela mudança
- [ ] Nenhuma dependência nova de serviço externo sem fallback ou circuit breaker
- [ ] Ingress/rotas novas documentadas se afetam DNS ou TLS

## Bloco 6: Qualidade de código

- [ ] Sem código comentado desnecessário
- [ ] Sem `console.log`, `dd()`, `dump()`, `var_dump()` esquecidos
- [ ] Funções e classes com responsabilidade única
- [ ] Nenhuma duplicação óbvia que deveria ser extraída
- [ ] Cobertura de casos de erro (try/catch onde relevante, responses de erro adequados)
- [ ] Convenções do time respeitadas (`.claude/conventions.md`)

## Bloco 7: Testes e validação

- [ ] Fluxo principal testado manualmente (ou automatizado)
- [ ] Casos de erro testados (input inválido, serviço indisponível, permissão negada)
- [ ] Rollback planejado caso o deploy precise ser revertido
- [ ] Feature flag necessária para release gradual (se aplicável)

---

## Verificação de known-issues

Após os blocos, verifique se algum item do `.claude/known-issues.md` é relevante para o que está sendo auditado. Se sim, liste quais e confirme se foram tratados.

## Registro em known-issues

Se a auditoria revelar um problema novo com potencial de ser recorrente, adicione em `.claude/known-issues.md`:

```markdown
## [Categoria — título curto]

Descrição: problema encontrado.
Contexto: onde tende a aparecer.
Solução: como prevenir ou corrigir.
Detectado em: [audit-code]
```

Informe o usuário: _"⚠️ Registrado em known-issues: [título]"_

## Parecer final

```
## Resumo da Auditoria

Críticos ❌: N
Atenção  ⚠️: N
OK       ✅: N
N/A      ➖: N

## Itens Críticos (se houver)
[lista numerada e acionável]

## Itens de Atenção (se houver)
[lista]

## Known Issues
[novos registros, ou "Nenhum novo problema registrado"]

## Veredito
✅ PRONTO PARA DEPLOY
⚠️ DEPLOY COM RESSALVAS — resolver atenções em seguida
❌ BLOQUEADO — resolver críticos antes do deploy
```
