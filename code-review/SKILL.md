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

## Stage 0: Clarification

Before reviewing, check if there is sufficient context to perform an accurate review.

Ask up to 3 targeted questions if any of the following are true:
- No description was provided AND no open files / diff / PR context is visible
- The described behavior contradicts what the code appears to do (needs confirmation of intent)
- The review scope is undefined (e.g., a very large diff with no indication of which areas matter most)

Do not ask style or preference questions. Only ask when missing context would produce an inaccurate review.

After asking, wait for the user's reply before proceeding to Stage 1.
If the user says "skip questions" or "just review", proceed immediately.

## Stage execution

By default, all stages run sequentially in one response.

The user can say "do only Stage 1 and 2", "stop after Stage 2", "skip to Stage 4". Respect this literally.

If during a stage a critical ambiguity appears (e.g., a pattern that could be intentional or a bug), flag it inline as:
> Question before continuing: [question]. If not answered, proceeding with conservative assumption: [stated assumption].

## Stage 1: Flow validation

Before analyzing the code (only if description was provided):

1. Understand the described flow
2. Identify logical gaps, uncovered cases, questionable approach
3. Issue verdict: ✅ Sound / ⚠️ With caveats / ❌ Problematic

Do not reject based on aesthetic preference — only real functional or security problems.

## Stage 2: Code analysis

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

If more than 3 critical items are found and the scope of change is unclear, pause after listing them and ask: "Found N critical items. Continue with Important/Suggestion items, or stop here to address these first?"

## Stage 3: Recurring issues detection

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

## Stage 4: Final verdict

**✅ APPROVED** — No critical issues
**⚠️ APPROVED WITH CAVEATS** — Can open PR, yellow items must be resolved
**❌ BLOCKED** — Critical items must be resolved before PR

## Output format

```
## Stage 1 — Flow Validation
[verdict / "No description provided"]

## Stage 2 — Code Analysis
### Critical 🔴
[list or "None"]
### Important 🟡
[list or "None"]
### Suggestions 🟢
[list or "None"]

## Stage 3 — Known Issues
[new items added / "No new issues recorded"]

## Stage 4 — Verdict
[✅ / ⚠️ / ❌] — justification in 1 line
```
