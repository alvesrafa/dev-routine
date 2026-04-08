---
name: code-review
description: Performs a complete code review of implementations. Use this skill when the user triggers /code-review, or when asking to review code, validate an implementation, or analyze a PR. Always use when the message starts with /code-review.
---

# Skill: /code-review

## Trigger

```
/code-review "description of what was implemented"
```

The description is optional but allows validating the flow before analyzing the code.

## Response guidelines

Be comprehensive but concise. Omit obvious explanations, get straight to the point. Save tokens without losing technical precision. Prefer short bullets to paragraphs.

## Reading project context

Read **if they exist**:

- `.claude/project.md` — stack, environments, modules
- `.claude/conventions.md` — mandatory team standards
- `.claude/architecture.md` — expected architectural patterns
- `.claude/known-issues.md` — to identify recurring issues

## Phase 1: Validation of described flow (if description provided)

Before analyzing the code:

1. Understand the described flow
2. Identify logical gaps, uncovered cases, questionable approach
3. Issue verdict: ✅ Sound / ⚠️ With caveats / ❌ Problematic

Do not reject based on aesthetic preference — only real functional or security problems.

## Phase 2: Code analysis

### 🔴 Critical — Blocks the PR

- Security vulnerabilities (SQL injection, exposed data, bypassable auth)
- Data loss (delete without soft delete when expected, accidental truncate)
- Race conditions in jobs/queues
- Migrations without `down()` or that lock large tables
- Hardcoded secrets or credentials
- Breaking API contract without versioning
- Logic that directly contradicts the stated requirement

### 🟡 Important — Must be fixed before merge

- Violation of team conventions (`.claude/conventions.md`)
- Missing error handling at critical points
- N+1 queries or heavy operations inside loops
- Job/command not idempotent when it should be
- Missing input validation on public endpoint
- Violation of project architectural patterns

### 🟢 Suggestion — Optional improvement

- Opportunity for reuse
- Confusing but functional naming
- Possible simplification without risk

## Phase 3: Detection of recurring issues

Check if any problem found already exists in `.claude/known-issues.md` or is new with recurring potential.

### Automatic known-issues update

If new and recurring, add to `.claude/known-issues.md`:

```markdown
## [Problem Category]

Description: problem found.
Context: where it tends to appear.
Solution: how to resolve or prevent.
Detected in: [code-review]
```

Inform: _"⚠️ Added to known-issues: [title]"_

If `.claude/known-issues.md` doesn't exist and there's something to record, create the file.

## Phase 4: Final verdict

**✅ APPROVED** — No critical issues  
**⚠️ APPROVED WITH CAVEATS** — Can open PR, yellow items must be resolved  
**❌ BLOCKED** — Critical items must be resolved before PR

## Output format

```
## Flow Validation
[verdict / "No description provided"]

## Critical Items 🔴
[list or "None"]

## Important Items 🟡
[list or "None"]

## Suggestions 🟢
[list or "None"]

## Known Issues
[new items added / "No new issues recorded"]

## Verdict
[✅ / ⚠️ / ❌] — justification in 1 line
```
