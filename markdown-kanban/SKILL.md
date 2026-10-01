---
name: markdown-kanban
description: Initialize and maintain a two-state Markdown Kanban workflow (Todo | Done) for AI-agent-driven software projects. Use when the user wants project planning, requirement management, task cards, markdown issue files, a kanban board, lightweight work tracking under docs/kanban, or session-based task discipline with definition-of-done gating.
---

# Markdown Kanban Skill

Use this skill to set up and operate a lightweight Markdown Kanban workflow inside any software repository, designed for AI-agent-driven development where one task is typically completed within one session.

The design is deliberately minimal:

- The board has exactly two columns: `Todo` and `Done`. No In Progress, no review/QA columns.
- `board.md` is only an index. Issue files carry all details and are the source of truth.
- `roadmap.md` holds long-term direction so future work survives across sessions.
- The only gate between `Todo` and `Done` is the card's Definition of Done checklist, verified by checkable artifacts — never by role-play.

## Directory layout

Create and maintain this structure at the project root:

```text
docs/kanban/
├── roadmap.md
├── board.md
├── issues/
│   ├── <ID>-<slug>.md
│   └── archive/        # optional: retired Done cards move here
└── templates/
    └── issue-template.md
```

- `docs/kanban/roadmap.md` — long-term goals, milestones, themes, candidate work.
- `docs/kanban/board.md` — index of current cards, two columns only.
- `docs/kanban/issues/` — one detailed Markdown file per card; `archive/` for retired Done cards.
- `docs/kanban/templates/issue-template.md` — the reusable card template.

If a repository already has another planning system, do not overwrite it blindly. Read it first and either integrate with it or ask the user how to proceed.

## Planning layers

Use two planning layers instead of turning every future idea into a detailed card immediately:

1. `roadmap.md` — long-term direction, milestones, epics, themes, candidate work. This prevents long-term goals from being lost when a new agent session starts.
2. `board.md` + `issues/` — executable work for the current and near-term cycles.

Do not create detailed cards for every possible future requirement at once; they are expensive to maintain and quickly go stale. Instead:

- Keep `roadmap.md` broad: themes, milestones, parking lot.
- Keep the board focused on the next 5–10 actionable items or the current 1–2 iterations.
- Convert a roadmap item into a card only when it becomes near-term work.
- Use discovery/spike cards for uncertain work before committing to implementation cards.

The board holds only executable work. Anything not yet actionable stays in `roadmap.md` — there is no Backlog column.

## Board

`board.md` has exactly two columns:

| Column | Meaning |
|---|---|
| `Todo` | Actionable work, including cards resumed after an interrupted session. |
| `Done` | Definition of Done fully verified. |

Hard rules:

- No other columns exist. "Blocked" and "waiting for user input" are fields on the card, not states.
- Moving a card means editing its row in `board.md` AND the `status` field in the issue file in the same operation. Never update one without the other.
- When `Done` grows past ~20 rows, move the oldest rows out of the board and move their issue files to `issues/archive/`. Do not delete issue files outright.

## Task ID convention

Use stable, type-based IDs:

| Prefix | Meaning |
|---|---|
| `F-` | Feature |
| `B-` | Bug |
| `D-` | Dashboard/UI surface |
| `R-` | Reports/data table |
| `S-` | Settings/configuration |
| `I-` | Infrastructure/integration |
| `Q-` | Quality/testing/refactor |
| `P-` | Packaging/release |
| `DOC-` | Documentation |

File name: `<ID>-<slug>.md`, for example `F-001-user-authentication.md` or `B-001-fix-startup-crash.md`. IDs are never reused. If no prefix clearly fits, use `F-` for user-facing functionality or `Q-` for internal quality work.

## Card fields (staged, not all at birth)

A card is filled in progressively; do not demand every field when creating it.

Required at creation (`Todo`):

- Front matter: `id`, `title`, `type`, `priority`, `size`, `status`, `created`, `updated`
- `Goal` — the outcome in one or two sentences
- `Acceptance Criteria` — outcome-focused, checkbox list

Required before `Done`:

- `Subtasks` checked, or explicitly deferred with a written reason
- `Test Cases` executed with recorded results, or a note explaining why tests do not apply
- `Verification` — related files, how to run/verify, actual results
- `Bugs` resolved or explicitly deferred
- `updated` date refreshed

Optional at any time: `Background`, `Dependencies`. `Development Log` becomes required once work starts.

### Priority

- `P0` — production/blocking issue; handle immediately.
- `P1` — required for the current milestone.
- `P2` — important, can follow P1 work.
- `P3` — nice-to-have or polish.

### Size

- `S` — one focused change.
- `M` — multiple steps or files, limited scope.
- `L` — cross-cutting or uncertain. An `L` card must be split into independently acceptable `S`/`M` cards before work starts.

### Bug severity

- `P0` — main flow broken, crash, data loss, or security issue. Blocks `Done`.
- `P1` — significant defect; fix or explicitly defer with rationale.
- `P2` — minor bug or polish; may be deferred.

## Definition of Done (the only gate)

A card moves `Todo` → `Done` only when all of the following are verifiable in its issue file:

1. Every subtask checkbox is checked, or deferred with a written reason.
2. Test cases have recorded results (pass/fail), or a note explains why tests do not apply.
3. Every acceptance criterion checkbox is checked.
4. No open `P0` bugs; `P1` bugs are fixed or deferred with rationale.
5. `Verification` records related files and how to run/verify.

Do not simulate Reviewer/QA/Maintainer roles to pass this gate. The gate is the checklist plus recorded artifacts, not role-play. If a step genuinely needs human judgment (design approval, product sign-off), set the `blocked` field with `owner: user`, ask the user, and leave the card in `Todo` until they answer.

## Blocked and waiting-on-user

Blocked is a field, not a state. A blocked card stays in `Todo` with its `blocked` front matter filled in:

- `reason` — what is blocking.
- `unblock` — what must be true to continue.
- `owner` — who acts next (usually the user or an external dependency).

Show the short reason in the card's `Blocked` board cell. Clear the field when resolved. A blocked card does not block the rest of the board.

## Session protocol

Tasks are typically completed within one session, so state management happens at session boundaries.

### Session start

1. Read `board.md` (and `roadmap.md` when planning).
2. Pick exactly one card: prefer one with checked subtasks but unchecked acceptance criteria (an interrupted session's leftover); otherwise the highest-priority card whose dependencies are resolved.
3. Work on it. Do not open a second card before the first is `Done` or carries a resume note in its log.

### Session close (always — including when stopping early or being interrupted)

1. Update subtask checkboxes to reflect reality.
2. Append a `Development Log` entry: what was done, key decisions, where it stopped, next step.
3. Make `status` in the issue file and the card's row in `board.md` agree.
4. Move to `Done` only if the Definition of Done passes; otherwise leave it in `Todo` with the resume note in the log.

## Default roadmap.md template

When initializing the workflow, create `docs/kanban/roadmap.md` like this and adapt milestones to the project:

```markdown
# Roadmap

This roadmap preserves long-term goals and milestone direction. Executable task cards live in `docs/kanban/issues/` and are tracked in `docs/kanban/board.md`.

## Planning Policy

- Do not create detailed issue cards for every possible future requirement at once.
- Keep detailed cards focused on the current 1–2 iterations or the next 5–10 actionable items.
- Use this roadmap for long-term themes, epics, and candidate work.
- Convert roadmap items into detailed task cards when they become near-term work.
- Use discovery/spike cards for uncertain work before creating implementation tasks.

## Milestones

### MVP / v0.1 - <milestone name>

Goal:
- <primary outcome>

Candidate tasks:
- <task or theme>

### v0.2 - <milestone name>

Goal:
- <primary outcome>

Candidate tasks:
- <task or theme>

### v1.0 - Stable release

Goal:
- <primary outcome>

Candidate tasks:
- <task or theme>

## Parking Lot

Future ideas that are not yet committed:

- <idea>
```

## Default board.md template

When initializing the workflow, create `docs/kanban/board.md` like this:

```markdown
# Kanban Board

Two columns only. Card details live in `docs/kanban/issues/<ID>-<slug>.md`.
Moving a card means updating its row here and `status` in the issue file in the same operation.

## Todo

| ID | Title | Priority | Size | Blocked | Dependencies |
|---|---|---|---|---|---|

## Done

| ID | Title | Priority | Done At |
|---|---|---|---|
```

## Default issue-template.md

When initializing the workflow, create `docs/kanban/templates/issue-template.md` like this:

```markdown
---
id: <ID>
title: <title>
type: feature          # feature | bug | refactor | test | docs | release | chore
priority: P1           # P0 | P1 | P2 | P3
size: M                # S | M | L
status: todo           # todo | done
created: YYYY-MM-DD
updated: YYYY-MM-DD
blocked: none          # none, or a mapping: reason / unblock / owner
---

## Goal

<The user-visible or engineering outcome, one or two sentences.>

## Background

<Optional. Why this task exists and relevant context.>

## Acceptance Criteria

- [ ] <outcome-focused criterion>

## Subtasks

- [ ] <subtask>

## Dependencies

- <dependency or `none`>

## Test Cases

### TC-001: <name>

Steps:
1. <step>

Expected:
- <expected result>

<!-- If tests do not apply, replace this section with a one-line reason. -->

## Development Log

### YYYY-MM-DD

- <what was done, decisions, where it stopped, next step>

## Bugs

| ID | Severity | Description | Status | Resolution |
|---|---|---|---|---|

## Verification

- Related files: `<path/to/file>`
- How to run/verify: <commands or steps>
- Results: <actual results, or reason not run>
```

## Operating procedure

### Initialize

1. Check whether `docs/kanban/` exists.
2. If missing, create `roadmap.md`, `board.md`, `issues/`, and `templates/issue-template.md` from the templates above.
3. If files exist, read them before editing; preserve existing content unless the user asks to rewrite.
4. If an existing board has more columns than `Todo | Done` (for example from an older version of this skill), collapse it: cards in Backlog/To Do go to `Todo`; cards in In Progress/Review/QA/Blocked go to `Todo` with their progress preserved in the issue file plus a `blocked` field if stuck; Done stays `Done`. Report the migration.
5. Report created or updated files.

### Create cards

1. Read `roadmap.md` and `board.md` first if they exist.
2. Prefer rolling refinement: detailed cards for near-term work only.
3. Assign a stable ID and slug; create the issue file from the template.
4. Fill only the creation-stage fields (front matter, Goal, Acceptance Criteria). Leave the remaining sections in place.
5. Add the row to `Todo` in `board.md` — the row and the front matter `status` must agree.
6. If the card came from a roadmap item, update the roadmap line to reference the new ID.

### Move Todo → Done

1. Walk the Definition of Done checklist against the issue file; tick every box and record evidence as you go.
2. If something fails, fix it first, or record it as a deferred `P1`/`P2` bug with rationale. Never tick a box that is not true.
3. Set `status: done` and refresh `updated` in the issue file, and move the row to `Done` with the date — both places, one operation.

### Handle blocked work

1. Fill the `blocked` field (reason / unblock / owner); keep the card in `Todo` and fill its `Blocked` board cell.
2. Tell the user what is needed to unblock.
3. Clear the field when resolved.

## Good practices

- `roadmap.md` is the durable home for long-term goals; never store long-term direction only in chat.
- Keep `board.md` lean — it is an index. Details and history belong in issue files.
- Record implementation surprises in the `Development Log` so a future session can resume safely.
- Keep acceptance criteria outcome-focused and test cases executable (steps plus expected result).
- Split `L` work into independently acceptable `S`/`M` cards instead of carrying it as one card.
- If you cannot verify something yourself, say so and ask the user — do not invent a pass.
