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

## Stage 0: Clarification

Before investigating, check if the description is actionable.

Ask up to 3 targeted questions if any of the following are true:
- No description was provided
- Expected vs. current behavior cannot be distinguished from the description
- The environment (local / staging / prod) is not determinable and matters for the hypotheses
- An error message or stack trace exists but was not shared

Only ask for facts that directly narrow the hypothesis space. No style or preference questions.

After asking, wait for the user's reply before proceeding to Stage 1.
If the user says "skip questions" or "just investigate", proceed with available context.

## Stage execution

By default, all stages run sequentially in one response.

The user can say "do only Stage 1 and 2", "stop after Stage 3", "skip to Stage 5". Respect this literally.

If a mid-stage ambiguity requires input, emit the completed portion of the current stage, then write:
> Paused at Stage N. [Question]. Reply to continue.

## Stage 1: Known issues check

Check `.claude/known-issues.md` first.

- If it matches something recorded: show the solution and ask if context changed
- If not: proceed to Stage 2

Output: _"Checking known-issues..."_ followed by result.

## Stage 2: Bug understanding

**Expected:** what should happen
**Current:** what is happening
**Environment:** infer from context (local / staging / prod)
**Frequency:** always / sometimes / specific condition

If after rephrasing the bug you identify a fundamental ambiguity that makes Stages 3–5 unreliable, pause here and ask the user to confirm before continuing.

## Stage 3: Root cause hypotheses

List ordered from most to least likely:

```
### Hypothesis N — [title]
Probability: High / Medium / Low
Cause: why this would explain the behavior
Verify: command or code snippet to confirm
```

If two hypotheses have equal probability and the distinction requires information only the user has (e.g., recent deploys, infra changes), pause after listing them and ask one targeted question before Stage 4.

## Stage 4: Diagnostic plan

Concrete steps in order of increasing cost:

1. Logs, database, queue (no code)
2. Code and flow
3. Reproduction in controlled environment

> Prefer commands executable **in pod terminal** for K8s environments.

## Stage 5: Fix plan

Only produce Stage 5 if either:
- At least one hypothesis from Stage 3 has High probability, OR
- The user has explicitly confirmed which hypothesis is correct.

If neither is true, output: "Stage 5 on hold — complete Stage 4 steps and share results to proceed."

- **Where:** file(s) and function(s)
- **What:** necessary change
- **Side effects:** what else might be affected
- **Test:** how to confirm the fix

## Stage 6: Register in known-issues

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
## Stage 1 — Known Issues
[Found with solution / Not found]

## Stage 2 — Bug Understanding
Expected: ...
Current: ...
Environment: ...
Frequency: ...

## Stage 3 — Hypotheses
[ordered list]

## Stage 4 — Diagnostic Plan
[numbered steps]

## Stage 5 — Fix Plan
[after diagnosis or high-confidence hypothesis / "On hold — see Stage 4"]

## Stage 6 — Known Issues Registration
[new record / "No registration needed"]
```
