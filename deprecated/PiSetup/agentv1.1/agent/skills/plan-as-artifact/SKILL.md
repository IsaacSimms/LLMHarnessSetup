---
name: plan-as-artifact
description: >
  Produce a structured markdown plan file and save it under docs/artifacts/ in the project.
  Use when the user says "write the plan", "build the plan artifact", "save the plan",
  "plan-as-artifact", or similar. Also applicable at the start of a planning session
  when the user wants a plan document produced before or after any design discussion.
  Can be used standalone or after a grill-me session. If a plan artifact seems clearly
  appropriate to the situation but the user hasn't asked, you may ask once whether they
  want one — do not push if they decline. For multi-session fog with many open decisions,
  prefer wayfinder (docs/plans/) instead of forcing a linear phase plan.
metadata:
  short-description: "Save a structured plan under docs/artifacts"
---

# plan-as-artifact

Produce a structured, model-optimized markdown plan from the current conversation and save
it under **`docs/artifacts/`** in the project. Audience: the human author and any **CLI coding
agent** (Grok Build, Claude Code, peers) that will execute or continue the plan in a later
session. Be specific, unambiguous, and self-contained — the model reading this file may have
no other context.

**Writing the plan is not permission to implement.** After save: report path, optionally show
TL;DR, and **stop** unless the user explicitly says to implement or continue.

---

## Routing (which skill)

| Situation | Skill | Location |
|-----------|--------|----------|
| Work **done**; durable record of what happened | `recap-thread` | `docs/recaps/` |
| Work **continues** in a fresh session | `thread-handoff` | `docs/handoffs/` |
| Decisions **settled**; implementable contract | `plan-as-artifact` | `docs/artifacts/` |
| Many open decisions / multi-session fog | `wayfinder` | `docs/plans/<map-slug>/` |
| Verify implement slice (build/tests/depth) | `check-work` | chat; optional `docs/checks/` |

If the effort is still foggy (major branches unresolved), **stop and offer wayfinder** instead of
writing a fake phase list. A handoff may **link** a plan path; it does not replace the plan.

---

## Resolve docs output directory

Shared by any skill that writes under `docs/`. This skill's subfolder is **`artifacts`**.

1. Start at the agent **cwd**.
2. If **`docs/` exists in cwd** → write under `{cwd}/docs/artifacts/`. Create `artifacts` if
   missing. **Do not ask.**
3. If no `docs/` in cwd → walk **up** looking for a **`docs/`** directory and/or a **git root**
   (`.git`).
4. If either is found → **ask the user** where to place the file. Offer clear defaults:
   - Nearest parent `docs/` → `{that-docs}/artifacts/`
   - Git root without `docs/` → create `{git-root}/docs/artifacts/`
   - If both exist and differ, list both and recommend the nearest `docs/`
5. If walk-up finds **nothing** → **ask**, default: create `{cwd}/docs/artifacts/`.

Create intermediate directories as needed when writing. Do **not** write plans to home-dir
folders as the primary target.

---

## Step 1: Resolve the filename

Filename convention (flat under `docs/artifacts/`):

**`{project-name-}{feature-or-task-slug}.md`**

- Infer `project-name` from the project root directory name when helpful; omit if redundant.
- Infer `feature-or-task-slug` from the problem or feature (kebab-case, concise).
- If the slug cannot be confidently inferred, ask before writing.
- **Stable name on update** — do not put the date in the filename (date lives in frontmatter /
  Revision Notes).

Examples:
- `codesmith-entra-external-id-wiring.md`
- `sccm-pxe-boot-remediation.md`
- `dashboard-refactor.md`

---

## Step 2: Write the plan file

Write to `{resolved-docs}/artifacts/{filename}`.

### Frontmatter (required)

```yaml
---
title: <short human title>
status: draft | ready | in-progress | done
updated: YYYY-MM-DD
project: <optional path or name if non-obvious>
---
```

Use `ready` when the plan is approved for execution but work has not started. Executing agents
move `status` to `in-progress` / `done` and refresh `updated` when they update Progress.

### Mandatory sections (always include)

#### TL;DR
Two to four sentences. What is being built or solved, why it matters, and the expected end
state. A model cold-starting from this file should understand full scope after this section only.

#### Problem Statement
What problem is being solved. What currently exists (if anything). What is broken, missing, or
suboptimal. Be concrete — fold current-state inventory here when it matters.

#### Requirements
Explicit, enumerated requirements. Each independently verifiable. Distinguish functional from
non-functional (performance, security, compatibility) where relevant.

#### Constraints
What is off the table, fixed, or non-negotiable. Technology, time/scope, system boundaries,
tooling. Under-specified constraints are the primary cause of execution drift.

#### Key Decisions
Decisions made during planning and the reasoning. Format:

> **Decision:** [what was decided]
> **Rationale:** [why this option over alternatives]
> **Alternatives considered:** [what was ruled out]

If this plan follows a grill-me session, surface **every** resolved branch here. Do not re-open
these in a later session unless the user explicitly reopens them.

#### Deferred
What is consciously **not** in this pass. If nothing is deferred, write exactly:
`Nothing deferred.`

#### Implementation Phases
Ordered phases. Each phase must include:
- A clear objective
- Ordered tasks
- Acceptance criteria — binary (met or not)

Size phases so each produces a reviewable, testable increment. Prefer interface-shaped work
over shallow task lists.

#### Progress
Checklist of phases/tasks for the **executing** agent to update as work completes. On create,
seed unchecked items from Implementation Phases (or leave a short empty checklist scaffold).
Updating Progress is part of “continue from this plan.”

#### Cold-start
Paste-ready opener for a new CLI session. Must instruct the receiving agent to:

1. Read **this plan file** (path given)
2. Honor **Constraints** and **Key Decisions** — do not re-litigate
3. Start at the first incomplete **Progress** item / phase
4. Only stop to ask if **Open Questions** (if present) block work
5. Not treat the plan file alone as permission to implement if the user has not said to build

Example shape:

> Read `docs/artifacts/{filename}` and continue from the plan. Honor Constraints and Key
> Decisions; do not re-open them. Start at the first incomplete Progress item. Confirm only if
> Open Questions block progress.

### Optional sections (include when they add value)

Do not add for completeness.

- **Open Questions** — unresolved items needed before or during execution
- **Dependencies** — external systems, APIs, packages, people
- **Testing Strategy** — when scope warrants explicit test planning (prefer TDD when applicable)
- **Rollback / Risk Notes** — high-risk or production-touching work
- **Out of Scope** — explicit exclusions (distinct from Deferred if helpful)
- **Reference Files** — key paths a model should read before starting

---

## Step 3: Confirm and report

After writing:

1. Full path of the saved plan
2. Point at the **Cold-start** block for future sessions
3. **Do not start implementing** unless the user explicitly asked to implement or continue

---

## Referencing the plan in a future session

If the user asks to load or continue a plan and no path is given:

1. Prefer `{resolved project}/docs/artifacts/` (resolve project the same way as write, or search
   obvious workspace roots)
2. Match by project name and task slug
3. If nothing in-repo: also search legacy **`~/.grok/plans/`** (and `~/.claude/plans/` if present).
   **Do not auto-migrate.** If a legacy match is found, confirm before using it
4. Confirm with the user: "I found `{path}` — is this the plan you want?"
5. If multiple matches, list them and ask

When continuing execution: update **Progress** and frontmatter `status` / `updated` as phases
complete. Do not remove historical Key Decisions.

---

## Updating an existing plan

- Preserve the filename unless scope changed significantly
- Append to **Key Decisions**; do not delete prior ones
- Add or extend **`## Revision Notes`** with a brief dated changelog entry
- Refresh `updated` in frontmatter

---

## Notes on quality

- Requirements are assertions, not aspirations: "must return within 200ms under normal load"
  not "should be fast."
- Constraints are as important as requirements.
- Acceptance criteria are binary.
- The plan is a contract between planner and executor. Ambiguity becomes bugs or rework.
- Prefer deep modules (small interface, rich behavior) when framing phases — not a laundry list
  of files to touch.
