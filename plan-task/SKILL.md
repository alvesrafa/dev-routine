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

Be comprehensive but concise. Omit obvious explanations, prefer lists and diagrams to paragraphs. Avoid unnecessary text.

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

Rephrase in 2–3 lines. List assumed premises if there's ambiguity.

If after rephrasing you identify a fundamental ambiguity that would make Stages 2–7 unreliable (e.g., two mutually exclusive interpretations of scope), pause here and ask the user to confirm before continuing.

## Stage 2: Implementation flowchart

Generate a Mermaid diagram representing the main task flow. Use `flowchart TD` for sequential tasks or `flowchart LR` for data/integration flows. Choose the type most suitable to the context.

Usage examples:

- Feature with multiple steps → `flowchart TD` with decisions and branches
- Integration between services → `flowchart LR` with systems as nodes
- Async job → sequence with queues and states

```mermaid
flowchart TD
    A[Start] --> B{Condition}
    B -->|Yes| C[Step 1]
    B -->|No| D[Alternative step]
    C --> E[End]
    D --> E
```

## Stage 3: Files and modules involved

| File/Folder                              | Action   | Reason      |
| ---------------------------------------- | -------- | ----------- |
| `app/Models/Foo.php`                     | Create   | New model   |
| `app/Http/Controllers/FooController.php` | Modify   | New endpoint|

## Stage 4: Dependencies and prerequisites

What needs to exist before starting: migrations, external services, permissions, feature flags.

## Stage 5: Implementation checklist

```
[ ] Step 1 — clear description
[ ] Step 2 — clear description
...
```

Each item should be small enough to be done and tested independently.

## Stage 6: Edge cases and risks

List only the relevant ones, by impact. If there's complex error flow, add a second Mermaid diagram.

## Stage 7: Acceptance criteria

How to know it's done. Verifiable, not subjective.

## Behavior with description

Use the description to:

- Identify type (feature / bug / refactor / infra)
- Adjust level of detail
- Detect modules by name and cross-reference with `.claude/architecture.md`
