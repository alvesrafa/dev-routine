---
name: fix-bug
description: Guides bug investigation and fixing in a structured way. Use this skill when the user triggers /fix-bug with a description, or when reporting an error, unexpected behavior, or asking to debug something. Always use when the message starts with /fix-bug.
---

# Skill: /fix-bug

## Trigger

```
/fix-bug "description of unexpected behavior"
```

The description is critical for investigation quality. Without it, ask before continuing.

## Response guidelines

Be comprehensive but concise. Omit obvious explanations, avoid repetition, get straight to the point. Save tokens without losing technical precision.

## Reading project context

Read **if they exist**:

- `.claude/project.md` — stack, environments, modules
- `.claude/known-issues.md` — **priority read**
- `.claude/architecture.md` — flows and dependencies
- `.claude/conventions.md` — team standards

## Phase 0: Check known-issues first

Check `.claude/known-issues.md` first.

- If it matches something recorded: show the solution and ask if context changed
- If not: proceed

## Phase 1: Bug understanding

**Expected:** what should happen  
**Current:** what is happening  
**Environment:** infer from context (local / staging / prod)  
**Frequency:** always / sometimes / specific condition

Ask maximum 2 questions if something is ambiguous.

## Phase 2: Root cause hypotheses

List ordered from most to least likely:

```
### Hypothesis N — [title]
Probability: High / Medium / Low
Cause: why this would explain the behavior
Verify: command or code snippet to confirm
```

## Phase 3: Diagnostic plan

Concrete steps in order of increasing cost:

1. Logs, database, queue (no code)
2. Code and flow
3. Reproduction in controlled environment

> Prefer commands executable **in pod terminal** for K8s environments.

## Phase 4: Fix plan

- **Where:** file(s) and function(s)
- **What:** necessary change
- **Side effects:** what else might be affected
- **Test:** how to confirm the fix

## Phase 5: Register in known-issues

If new bug with recurring potential, add to `.claude/known-issues.md`:

```markdown
## [Category — title]

Description: behavior and root cause.
Context: where it tends to appear.
Solution: how to fix.
Detected in: [fix-bug]
```

Inform: _"⚠️ Recorded in known-issues: [title]"_

## Output format

```
## Known Issues
[Found with solution / Not found]

## Bug Understanding
Expected: ...
Current: ...
Environment: ...

## Hypotheses
[ordered list]

## Diagnostic Plan
[numbered steps]

## Fix Plan
[after diagnosis or high confidence in hypothesis]

## Known Issues
[new record / "No registration needed"]
```
