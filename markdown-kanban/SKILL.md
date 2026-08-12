---
name: markdown-kanban
description: Initialize and maintain a reusable Markdown Kanban requirements workflow for software projects. Use when the user wants project planning, requirement management, task cards, markdown issue files, Kanban boards, lightweight sprint tracking, review/QA/acceptance workflow, or asks to manage work under docs/kanban.
---

# Markdown Kanban Skill

Use this skill to set up and operate a project-agnostic Markdown Kanban workflow inside any software repository.

The goal is a lightweight requirements management system that works well for AI-agent-driven development: every meaningful feature, bug, refactor, release task, or QA task has a persistent Markdown card, and the board shows status at a glance.

## Directory layout

Create and maintain this structure at the project root:

```text
docs/kanban/
├── roadmap.md
├── board.md
├── issues/
│   └── <ID>-<slug>.md
└── templates/
    └── issue-template.md
```

- `docs/kanban/roadmap.md` preserves long-term goals, milestones, themes, and future candidate work across sessions.
- `docs/kanban/board.md` is the source of truth for board columns and card status.
- `docs/kanban/issues/` stores one detailed Markdown issue file per task.
- `docs/kanban/templates/issue-template.md` stores the reusable issue template.

If a repository already has another planning system, do not overwrite it blindly. Read it first and either integrate with it or ask the user how to proceed.

## Planning layers

Use two planning layers instead of turning every future idea into a detailed card immediately:

1. `roadmap.md` — long-term direction, milestones, epics, themes, and candidate work. This prevents long-term goals from being lost when a new agent session starts.
2. `board.md` + `issues/` — executable work for the current and near-future cycles.

Do not create detailed issue cards for every possible future requirement at once. Detailed cards are expensive to maintain and quickly become stale. Prefer this rule:

- Keep `roadmap.md` broad enough to cover the long-term product direction.
- Keep detailed issue cards focused on the current 1–2 iterations or the next 5–10 actionable items.
- Convert roadmap items into detailed issue cards only when they become near-term work or need concrete acceptance criteria.
- Use discovery/spike cards for uncertain work before committing to implementation cards.

## Board columns

Use these project-level columns:

1. `Backlog` — requirement pool, not committed to the current cycle.
2. `To Do` — selected for the current sprint/cycle.
3. `In Progress` — active development.
4. `In Review` — implementation complete, under code/design review.
5. `Testing / QA` — QA is executing test cases and recording bugs.
6. `Blocked` — cannot proceed due to a dependency or decision.
7. `Done` — accepted by product/maintainer.

## Task ID convention

Use stable task IDs. Prefer type-based prefixes:

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

Examples:

- `F-001-user-authentication.md`
- `B-001-fix-startup-crash.md`
- `R-001-daily-usage-table.md`
- `P-001-windows-publish.md`

If no prefix clearly fits, use `F-` for user-facing functionality or `Q-` for internal quality work.

## Required fields for every task card

Each task issue file must include:

- `ID`
- `Title`
- `Type`
- `Priority`
- `Status`
- `Size`
- `Milestone`
- `Developer`
- `Reviewer`
- `QA`
- `Maintainer`
- `Goal`
- `Background`
- `Scope`
- `Subtasks`
- `Acceptance Criteria`
- `Test Cases`
- `Related Files`
- `Dependencies`
- `Development Log`
- `Review Notes`
- `QA Notes`
- `Bugs`
- `Acceptance Notes`

### Priority

Use priority for urgency/importance, not effort:

- `P0` — production/blocking issue; must be handled immediately.
- `P1` — required for the current milestone.
- `P2` — important but can be scheduled after P1 work.
- `P3` — nice-to-have or polish.

### Size

Use size for rough complexity:

- `S` — small, usually one focused change.
- `M` — medium, multiple steps/files but limited scope.
- `L` — large, cross-cutting or uncertain; consider splitting.

### Bug severity

Record QA bugs as:

- `P0` — blocks the main flow, crash/data loss/security issue; must be fixed before Done.
- `P1` — significant defect; fix or explicitly defer with rationale.
- `P2` — minor bug/polish; may be deferred.

## Agent role simulation

The agent may act in multiple workflow roles for solo or small projects:

- `Developer` — implements and writes/updates tests where applicable.
- `Reviewer` — reviews changes, checks risk, and records comments.
- `QA` — executes documented test cases and records bugs.
- `Maintainer` / `Product Manager` — verifies acceptance criteria and moves work to Done.

Be explicit that this is a process simulation and not independent third-party review. It helps enforce discipline, but it does not replace a real human reviewer or QA tester when those are required.

## Strict status rules

### Backlog → To Do

Move a task to `To Do` when it is selected for the current work cycle and has enough detail to start.

Before moving to `To Do`, ensure:

- Goal is clear.
- Acceptance Criteria exist.
- Test Cases exist or the issue explains why tests are not applicable.
- Dependencies are listed.

### To Do → In Progress

Move a task to `In Progress` when development starts.

When entering `In Progress`:

- Set `Developer`.
- Add a `Development Log` entry with date and intended approach.
- Keep `board.md` and the issue file in sync.

### In Progress → In Review

Move to `In Review` only when implementation is complete.

Developer responsibilities:

- Complete the subtasks or record deferred items.
- Add/update unit tests where applicable.
- Run relevant checks and record results.
- Add a review summary containing:
  - Modification points.
  - How to run/test.
  - Impact area.
  - Regression risk.

If there is no actual pull request, record a `PR-equivalent Review Summary` in the issue file.

### In Review → Testing / QA

Reviewer responsibilities:

- Check the implementation against the goal and acceptance criteria.
- Check code clarity and maintainability.
- Check likely regression areas.
- Record `Approved` or `Request Changes` in `Review Notes`.

At least one reviewer approval is required before moving to `Testing / QA`. In solo-agent workflows, the agent can switch to `Reviewer` role and explicitly mark this as simulated review.

### Testing / QA → Done or In Progress

QA responsibilities:

- Execute each documented test case.
- Record pass/fail results in `QA Notes`.
- Record bugs under `Bugs` with severity `P0/P1/P2`.

Rules:

- All `P0` bugs must be fixed before Done.
- `P1` bugs must be fixed or explicitly deferred with maintainer/product rationale.
- `P2` bugs may be fixed or deferred.
- When testing passes, write `QA Passed` in `QA Notes`.

If bugs require code changes, move the card back to `In Progress` and continue the cycle.

### Testing / QA → Done

Maintainer/Product responsibilities:

- Verify every Acceptance Criterion.
- Check QA status.
- Record final acceptance notes.
- Move the task to `Done` only after acceptance passes.

In solo-agent workflows, the agent may simulate Maintainer/Product acceptance, but must say it is simulated.

### Any state → Blocked

Move to `Blocked` when progress cannot continue.

Record:

- Blocker description.
- Owner of the unblock action.
- Next decision/action needed.
- Date blocked.

Move back to the appropriate column once unblocked.

## Default roadmap.md template

When initializing the workflow, create `docs/kanban/roadmap.md` like this and adapt milestones to the project:

```markdown
# Roadmap

This roadmap preserves long-term goals and milestone direction. Detailed executable task cards live in `docs/kanban/issues/` and are tracked in `docs/kanban/board.md`.

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

This board tracks project requirements and implementation tasks. Detailed task cards live in `docs/kanban/issues/`.

## Backlog

| ID | Title | Priority | Size | Owner | Milestone | Dependencies |
|---|---|---|---|---|---|---|

## To Do

| ID | Title | Priority | Size | Owner | Milestone | Dependencies |
|---|---|---|---|---|---|---|

## In Progress

| ID | Title | Priority | Size | Owner | Milestone | Started |
|---|---|---|---|---|---|---|

## In Review

| ID | Title | Priority | Size | Developer | Reviewer | Review Status |
|---|---|---|---|---|---|---|

## Testing / QA

| ID | Title | Priority | Size | Developer | QA | QA Status |
|---|---|---|---|---|---|---|

## Blocked

| ID | Title | Priority | Size | Owner | Blocker | Blocked Since |
|---|---|---|---|---|---|---|

## Done

| ID | Title | Priority | Size | Owner | Milestone | Done At |
|---|---|---|---|---|---|---|
```

## Default issue-template.md

When initializing the workflow, create `docs/kanban/templates/issue-template.md` like this:

```markdown
# <ID>: <Title>

## Metadata

- ID: <ID>
- Title: <Title>
- Type: Feature | Bug | Refactor | Test | Docs | Release | Chore
- Priority: P0 | P1 | P2 | P3
- Status: Backlog | To Do | In Progress | In Review | Testing / QA | Blocked | Done
- Size: S | M | L
- Milestone: <milestone>
- Developer: <name or TBD>
- Reviewer: <name or TBD>
- QA: <name or TBD>
- Maintainer: <name or TBD>
- Created: YYYY-MM-DD
- Updated: YYYY-MM-DD

## Goal

Describe the user-visible or engineering goal in one or two paragraphs.

## Background

Explain why this task exists, what problem it solves, and any relevant context.

## Scope

### In Scope

- [ ] <included work>

### Out of Scope

- <explicitly excluded work>

## Subtasks

- [ ] <subtask 1>
- [ ] <subtask 2>

## Acceptance Criteria

- [ ] <criterion 1>
- [ ] <criterion 2>

## Test Cases

### TC-001: <test case name>

Steps:
1. <step>
2. <step>

Expected:
- <expected result>

## Related Files

- `<path/to/file>`

## Dependencies

- <dependency or `None`>

## Development Log

### YYYY-MM-DD

- <implementation note, problem encountered, or decision made>

## Review Notes

- Status: Not Reviewed | Approved | Request Changes
- Reviewer: <name>
- Notes:
  - <note>

## QA Notes

- Status: Not Started | In Progress | QA Passed | QA Failed
- QA: <name>
- Results:
  - <result>

## Bugs

| ID | Severity | Description | Status | Resolution |
|---|---|---|---|---|

## Acceptance Notes

- Status: Not Accepted | Accepted | Rejected
- Maintainer: <name>
- Notes:
  - <note>
```

## Operating procedure

### Initialize Kanban

When the user asks to initialize this workflow:

1. Check whether `docs/kanban/` exists.
2. If missing, create:
   - `docs/kanban/roadmap.md`
   - `docs/kanban/board.md`
   - `docs/kanban/issues/`
   - `docs/kanban/templates/issue-template.md`
3. If files already exist, read them before editing.
4. Preserve existing roadmap items and tasks unless the user explicitly asks to rewrite.
5. Report created or updated files.

### Create new task cards

When creating task cards:

1. Read `docs/kanban/roadmap.md` and `docs/kanban/board.md` first if they exist.
2. Prefer rolling refinement: create detailed cards for near-term work, not every long-term roadmap idea.
3. Assign a stable ID and slug.
4. Create one issue file under `docs/kanban/issues/`.
5. Add the task to the requested board column, usually `Backlog` or `To Do`.
6. Ensure the issue file and `board.md` status match.
7. Include realistic Acceptance Criteria and Test Cases. If tests do not apply, explain why.
8. If a detailed card came from a roadmap item, update the roadmap candidate task line to reference the new ID.

### Move tasks between statuses

When moving a task:

1. Read `board.md` and the task issue file.
2. Check the status transition rules.
3. Update `board.md` by moving the row to the target column.
4. Update the issue file metadata `Status` and `Updated` date.
5. Add a log entry explaining the transition.
6. If the transition requires review, QA, or acceptance notes, add them.

### During implementation

When working on a task managed by this Kanban:

1. Move the card to `In Progress` if it is not already there.
2. Implement the work.
3. Update the task's Development Log with key decisions and problems.
4. Add/update tests where applicable.
5. Run relevant checks.
6. Move to `In Review` with a PR-equivalent review summary.
7. Simulate Reviewer if requested or if this is a solo-agent workflow.
8. Move to `Testing / QA` after approval.
9. Execute test cases as QA and record results/bugs.
10. Move to `Done` only after acceptance criteria are verified.

## Good practices

- Keep `roadmap.md` as the durable home for long-term goals and milestone direction.
- Keep `board.md` concise; put task details in issue files.
- Do not bury critical requirements only in chat. Persist near-term work in issue files and long-term direction in `roadmap.md`.
- Do not create detailed cards for every future idea at once; use rolling refinement from roadmap to issue cards.
- Prefer splitting `L` tasks into smaller `S` or `M` tasks.
- Keep Acceptance Criteria outcome-focused.
- Keep Test Cases executable: steps plus expected results.
- Record important implementation surprises in `Development Log` so future agents can continue safely.
- If the agent is simulating roles, label entries clearly, for example: `Reviewer: ZCode (simulated)`.
