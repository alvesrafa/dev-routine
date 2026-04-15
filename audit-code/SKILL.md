---
name: audit-code
description: Performs a complete security, quality, and infrastructure audit before deploying to production. Use this skill when the user triggers /audit-code, or when asking to audit, verify if something is ready for production, or do a quality checklist. Always use when the message starts with /audit-code.
---

# Skill: /audit-code

## Trigger

The user triggers with:

```
/audit-code "description of what will be audited"
```

The description is optional. Without it, the audit is performed on the available general context (open files, recent diff, or conversation).

## Reading project context

Before any analysis, search for and read the following files **if they exist**:

- `.claude/project.md` — stack, environments, modules
- `.claude/conventions.md` — team standards to check compliance
- `.claude/architecture.md` — expected infrastructure and code patterns
- `.claude/known-issues.md` — check if any documented problems are present

## Stage 0: Clarification

Before auditing, check if the scope is clear enough to apply the right stages.

Ask up to 3 targeted questions if any of the following are true:
- No description was provided AND no diff/files are visible in context
- The deployment target is unknown (K8s, serverless, bare VM) and Stage 5 (Infrastructure) is relevant
- The audit intent is unclear — full pre-deployment audit vs. targeted review of a specific change

Do not ask about code style or team preferences. Only ask when missing context would cause entire stages to be skipped or incorrectly applied.

After asking, wait for the user's reply before proceeding to Stage 1.
If the user says "skip questions" or "audit everything", proceed with all 7 stages.

## Stage execution

By default, all 7 stages execute in one response.

The user can restrict this: "audit only Stages 1 and 2", "stop after Stage 3", "skip to Stage 5". Respect this literally.

If the user asks for a targeted audit (e.g., "audit the jobs only"), map their request to the relevant stage(s) and run only those.

For each item in every stage, mark:

- ✅ OK
- ⚠️ Attention (doesn't block, but should be reviewed)
- ❌ Critical (blocks deployment)
- ➖ Not applicable

---

## Stage 1: Security

- [ ] No credentials, secrets, or API keys hardcoded in code
- [ ] Sensitive environment variables read from `.env` / K8s secrets, never committed
- [ ] Public endpoints with input validation (Form Request / middleware)
- [ ] Authentication and authorization checked on new or modified routes
- [ ] File uploads with type and size validation
- [ ] Queries built with bindings (never direct interpolation of user input)
- [ ] CORS correctly configured for the environment

If any Security item is marked ❌ Critical, after completing Stage 1 ask:
"Found critical security issue(s). Proceed with remaining audit stages, or stop here to address security first?"

## Stage 2: Database

- [ ] Every migration has functional `down()`
- [ ] Migrations that alter large tables have strategy to avoid lock (e.g., `ADD COLUMN` with null default)
- [ ] New indexes created for fields used in frequent `WHERE`, `JOIN`, or `ORDER BY`
- [ ] Soft deletes implemented where data should not be permanently lost
- [ ] No `DB::statement` with DDL without transaction protection or environment check
- [ ] Seeds and factories don't run in production (guard with `App::environment`)

## Stage 3: Performance

- [ ] No N+1 queries (relations eager-loaded where necessary)
- [ ] Heavy operations outside request cycle (jobs, commands)
- [ ] Jobs with appropriate chunk size for large volumes
- [ ] Cache used where data is read frequently and changes infrequently
- [ ] Pagination on listings that may return many records

## Stage 4: Queues and Jobs

- [ ] Jobs implement retry with appropriate backoff
- [ ] Jobs are idempotent (reprocessing causes no side effect)
- [ ] Job failures logged with sufficient context for debugging
- [ ] Correct queue configured (not everything on `default`)
- [ ] Job timeout explicitly set when operation can be long

## Stage 5: Infrastructure and K8s

- [ ] Required environment variables documented (`.env.example` updated)
- [ ] New secrets added to Key Vault and corresponding K8s manifest
- [ ] Pod CPU/memory resources adequate for expected load
- [ ] Health checks (liveness/readiness) not broken by change
- [ ] No new external service dependency without fallback or circuit breaker
- [ ] New Ingress/routes documented if they affect DNS or TLS

## Stage 6: Code quality

- [ ] No unnecessary commented code
- [ ] No forgotten `console.log`, `dd()`, `dump()`, `var_dump()`
- [ ] Functions and classes with single responsibility
- [ ] No obvious duplication that should be extracted
- [ ] Error cases covered (try/catch where relevant, appropriate error responses)
- [ ] Team conventions respected (`.claude/conventions.md`)

## Stage 7: Tests and validation

- [ ] Main flow tested manually (or automated)
- [ ] Error cases tested (invalid input, service unavailable, permission denied)
- [ ] Rollback planned if deployment needs reverting
- [ ] Feature flag needed for gradual rollout (if applicable)

---

## Check known-issues

After the stages, check if any item in `.claude/known-issues.md` is relevant to what is being audited. If yes, list which ones and confirm if they were addressed.

## Register in known-issues

If the audit reveals a new problem with potential to be recurring, add to `.claude/known-issues.md`:

```markdown
## [Category — short title]

Description: problem found.
Context: where it tends to appear.
Solution: how to prevent or fix.
Detected in: [audit-code]
```

Inform the user: _"⚠️ Recorded in known-issues: [title]"_

## Final verdict

```
## Audit Summary

| Stage | Name | ❌ | ⚠️ | ✅ | ➖ |
|-------|------|----|----|----|-----|
| 1 | Security | N | N | N | N |
| 2 | Database | N | N | N | N |
| 3 | Performance | N | N | N | N |
| 4 | Queues/Jobs | N | N | N | N |
| 5 | Infrastructure | N | N | N | N |
| 6 | Code Quality | N | N | N | N |
| 7 | Tests | N | N | N | N |
| **Total** | | **N** | **N** | **N** | **N** |

## Critical Items (if any)
[numbered and actionable list]

## Attention Items (if any)
[list]

## Known Issues
[new records, or "No new issues recorded"]

## Verdict
✅ READY FOR DEPLOYMENT
⚠️ DEPLOY WITH CAVEATS — resolve attention items after
❌ BLOCKED — resolve critical items before deployment
```
