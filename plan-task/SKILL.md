---
name: plan-task
description: Plans development tasks in a structured way for teams. Use this skill when the user triggers /plan-task with a description, or when asking to plan, detail, or break down a development task into steps. Always use when the message starts with /plan-task.
---

# Skill: /plan-task

## Trigger

```
/plan-task "task description"
```

Without a description, ask before continuing.

## Response guidelines

Be comprehensive but concise. Prefer tables and lists over paragraphs. Omit obvious explanations. No flowcharts.

## Reading project context

Read **if they exist**:

- `.claude/project.md` — stack, environments, modules
- `.claude/conventions.md` — team standards
- `.claude/architecture.md` — technical decisions
- `.claude/known-issues.md` — known pitfalls

If they don't exist, inform and continue with generic context.

## Stage 0: Clarification

Before producing the plan, check if the description is sufficient.

Ask up to 3 targeted questions if any of the following are true:
- The task type cannot be determined (feature / bug / refactor / infra)
- The affected module or service is not identifiable from the description or `.claude/architecture.md`
- The scope boundary is ambiguous (e.g., "improve performance" with no target component named)

Do not ask about implementation preferences — only ask for facts needed to scope the plan correctly.

After asking, wait for the user's reply before proceeding to Stage 1.
If the description is clear enough, skip Stage 0 silently and proceed.

## Stage execution

By default, all stages run sequentially in one response.

The user can say "do only Stage 1", "stop after Stage 3", "skip to Stage 5". Respect this literally.

If a mid-stage ambiguity requires input, emit the completed portion of the current stage, then write:
> Paused at Stage N. [Question]. Reply to continue.

## Stage 1: Task understanding

Rephrase in 2–3 lines. List assumed premises as a table if there's ambiguity.

| Premise | Assumed value |
|---------|--------------|
| Task type | feature / bug / refactor / infra |
| Target module | identified name or "unknown" |

If after rephrasing you identify a fundamental ambiguity that would make the remaining stages unreliable, pause here and ask the user to confirm before continuing.

## Stage 2: Files and modules involved

| File / Folder | Action | Reason |
|---------------|--------|--------|
| `path/to/file` | Create / Modify / Delete | Short reason |

## Stage 3: Dependencies and prerequisites

| Item | Type | Required before |
|------|------|----------------|
| Migration X | DB | Step 2 |
| Service Y running | Infra | Step 1 |
| Feature flag Z | Config | Step 3 |

If no dependencies exist, write: _No external dependencies identified._

## Stage 4: Implementation checklist

```
[ ] Step 1 — clear description
[ ] Step 2 — clear description
...
```

Each item should be small enough to be done and tested independently.

## Stage 5: Edge cases and risks

| Scenario | Impact | Mitigation |
|----------|--------|------------|
| Short description | High / Medium / Low | Short mitigation |

## Stage 6: Acceptance criteria

| Criterion | How to verify |
|-----------|--------------|
| Feature works end-to-end | Manual test or automated test description |
| No regressions | Existing test suite passes |

## Final output: Ready-to-use prompt

After completing all stages, output the ready-to-use prompt between the `START COPY` and `END COPY` markers. Nothing else between the markers.

**Rules for building the prompt:**

1. **Always fill `<context>` with real data** — task summary from Stage 1, file list from Stage 2, dependencies from Stage 3, risks from Stage 5. Never write placeholder text like "Paste here".
2. **Omit sections with no data** — if Stage 3 has no dependencies, omit that block from `<context>`. If Stage 5 has no risks, omit it too.
3. **Omit `<history>` block** — this is a fresh plan, there is no conversation history. Remove the entire `Conversation history: <history>…</history>` block.
4. **Omit `<request>` block** — same reason. Remove `Current request: <request>…</request>`.
5. **Omit `<response>` / `[FINAL ANSWER]`** — these are only useful in multi-turn structured sessions. For an implementation kick-off prompt, remove them.
6. **Omit `<example>` blocks** — do not include generic example placeholders.
7. **Keep the Rules section** — always include it, and customize based on stack conventions from `.claude/conventions.md` if it exists.
8. After the `END COPY` marker, add a one-line suggested implementation order based on the checklist.

**Output format:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ START COPY ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

You are [senior dev role inferred from the stack]. Your job is to implement the task described below. You are helping [user type] in [environment].

Use this tone: precise, direct, no filler.

Reference this material when answering:
<context>
[TASK SUMMARY from Stage 1]

[FILE LIST from Stage 2]

[DEPENDENCIES from Stage 3 — omit block if none]

[RISKS from Stage 5 — omit block if none]
</context>

Rules:
- Touch only the files listed above unless a deviation is explicitly justified
- Each implementation step must be independently testable before moving to the next
- If unsure about scope, ask before implementing — do not assume
[- Additional rules from .claude/conventions.md if it exists]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ END COPY ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> Prompt ready. Copy everything between the markers and trigger when you want to start implementation.
> Suggested order: [e.g. "tackle backend steps B-1 → B-N first (run tests after each migration), then frontend F-1 → F-N"]

## Behavior with description

Use the description to:

- Identify type (feature / bug / refactor / infra)
- Adjust level of detail
- Detect modules by name and cross-reference with `.claude/architecture.md`
