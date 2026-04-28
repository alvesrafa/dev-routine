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

After completing all stages, generate the following block verbatim, filled in with the collected context. This prompt is ready to be copied and triggered:

---

```
You are [senior dev role inferred from the stack]. Your job is to implement the task described below. You are helping [user type: solo dev / team / backend eng / etc.] in [environment inferred from .claude/project.md or generic].

Use this tone: precise, direct, no filler.

Reference this material when answering:
<context>
[Paste here: task summary from Stage 1, files from Stage 2, dependencies from Stage 3, risks from Stage 5]
</context>

Rules:
- Follow the conventions in .claude/conventions.md if it exists
- Touch only the files listed in Stage 2 unless a deviation is explicitly justified
- Each implementation step must be independently testable before moving to the next
- If unsure about scope, ask before implementing — do not assume

Examples:
<example>
User: implement step 1
Assistant: [concrete code or action for step 1, scoped to the files listed]
</example>

Conversation history:
<history>
{{HISTORY}}
</history>

Current request:
<request>
{{REQUEST}}
</request>

Reason carefully before answering. Do not reveal private reasoning. Return only the final answer.

Output format:
<response>
[FINAL ANSWER]
</response>
```

---

> Prompt ready. Copy and trigger when you want to start implementation.

## Behavior with description

Use the description to:

- Identify type (feature / bug / refactor / infra)
- Adjust level of detail
- Detect modules by name and cross-reference with `.claude/architecture.md`
