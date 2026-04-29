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

After completing all stages, output **one single copy block** with everything the implementer needs. There must be exactly one `START COPY` marker and one `END COPY` marker in the entire response — never inside the context, never duplicated.

**Rules for building the prompt:**

1. **One block only** — everything goes between the two markers. Do not split into multiple copy blocks.
2. **Fill `<context>` with ALL stages** — include task summary (Stage 1), file list (Stage 2), dependencies (Stage 3), full checklist (Stage 4), risks (Stage 5), acceptance criteria (Stage 6). Condense; remove headers like "Stage N:" — just the data.
3. **Omit sections with no data** — if Stage 3 has no dependencies, omit that section from `<context>`. Same for Stage 5 with no risks.
4. **Keep the Rules section** — always include it, customized with conventions from `.claude/conventions.md` if it exists.
5. **No placeholder text** — never write "Paste here" or "[...]". Every field must contain real data from the plan.
6. **No flowcharts, no `<history>`, no `<request>`, no `<example>` blocks** — remove all of these.

**Output format — emit this exactly once at the end:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ START COPY ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

You are [senior dev role inferred from the stack]. Your job is to implement the task described below.

Use this tone: precise, direct, no filler.

<context>
TASK SUMMARY:
[2–3 line summary from Stage 1]

CURRENT STATE (verified by code inspection):
[Key facts about the existing code that the implementer must know — nullable constraints, missing models, current type values, etc.]

FILES TO CREATE/MODIFY:
[Each file on its own line: path (Action) — reason]

DEPENDENCIES:
[Each dependency: item — type — must be done before X]
[Omit entire section if none]

IMPLEMENTATION CHECKLIST:
[Full checklist from Stage 4, grouped by area (BACKEND / FRONTEND / etc.)]

RISKS:
[Each risk: scenario — impact — mitigation]
[Omit entire section if none]

ACCEPTANCE CRITERIA:
[Each criterion: what — how to verify]
</context>

Rules:
- Touch only the files listed above unless a deviation is explicitly justified in a comment
- Each implementation step must be independently testable before moving to the next
- If unsure about scope, ask before implementing — do not assume
[- Additional rules from .claude/conventions.md if it exists]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ END COPY ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

After the `END COPY` marker, output these two lines and nothing else:

> Suggested order: [one sentence — e.g. "B-1 → B-9 first (run migrations + tests after each), then F-1 → F-10"]
> Recommended: start a new chat and paste the block above as the first message — keeps implementation context clean and avoids token overhead from this planning session.

## Behavior with description

Use the description to:

- Identify type (feature / bug / refactor / infra)
- Adjust level of detail
- Detect modules by name and cross-reference with `.claude/architecture.md`
