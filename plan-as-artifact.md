---
name: plan-as-artifact
description: >
  Produce a structured markdown plan file and save it to ~/.claude/plans/.
  Use when the user says "write the plan", "build the plan artifact", "save the plan",
  "plan-as-artifact", or similar. Also applicable at the start of a planning session
  when the user wants a plan document produced before or after any design discussion.
  Can be used standalone or after a grill-me session. If a plan artifact seems clearly
  appropriate to the situation but the user hasn't asked, you may ask once whether they
  want one — do not push if they decline.
allowed-tools: Read, Write, Bash(mkdir:*), Bash(ls:*)
---

# plan-as-artifact

Produce a structured, model-optimized markdown plan file from the current conversation
context and save it to `~/.claude/plans/`.

---

## Step 1: Resolve the filename

Filename convention: `{project-name}-{feature-or-task-slug}.md`

- Infer `project-name` from the current working directory name (the project root Claude
  Code was launched from).
- Infer `feature-or-task-slug` from the problem or feature being discussed (kebab-case, concise).
- If either cannot be confidently inferred, ask the user before proceeding.

Examples:
- `pairedprogrammer-clerk-auth-flow.md`
- `sccm-pxe-boot-remediation.md`
- `myapp-dashboard-refactor.md`

---

## Step 2: Ensure the plans directory exists

```bash
mkdir -p ~/.claude/plans
```

---

## Step 3: Write the plan file

Use the Write tool to write to `~/.claude/plans/{filename}`.

The plan is written for **two audiences**: the human author and any AI model that will
execute or continue work on this plan in a future session. Write with both in mind.
Be specific, unambiguous, and self-contained — the model reading this file may have
no other context.

### Mandatory sections (always include)

#### TL;DR
Two to four sentences. What is being built or solved, why it matters, and the expected
end state. A model cold-starting from this file should understand the full scope after
reading only this section.

#### Problem Statement
What problem is being solved. What currently exists (if anything). What is broken,
missing, or suboptimal. Be concrete — avoid vague framing.

#### Requirements
Explicit, enumerated requirements. Each requirement should be independently verifiable.
Distinguish functional requirements from non-functional (performance, security,
compatibility, etc.) where relevant.

#### Constraints
What is off the table, fixed, or non-negotiable. Technology constraints, time/scope
limits, existing system boundaries that cannot be changed, team or tooling restrictions.
This section prevents the executing model from proposing solutions that violate known
boundaries.

#### Key Decisions
Decisions that were made during planning and the reasoning behind them. Format:

> **Decision:** [what was decided]
> **Rationale:** [why this option was chosen over alternatives]
> **Alternatives considered:** [what was ruled out]

If this plan follows a grill-me session, surface every resolved branch here.

#### Implementation Phases
Ordered phases. Each phase must include:
- A clear objective for the phase
- Ordered tasks within the phase
- Acceptance criteria — how the model or developer knows the phase is complete

Phases should be sized so that each one produces a reviewable, testable increment.

### Optional sections (include when applicable)

The model should assess the problem domain and add sections where they add real value.
Do not add sections for completeness — only add them if they carry information a human
or model would act on.

Examples of optional sections:
- **Open Questions** — unresolved items that need a decision before or during execution
- **Dependencies** — external systems, APIs, packages, or people this work depends on
- **Testing Strategy** — if the scope warrants explicit test planning
- **Rollback / Risk Notes** — if the work is high-risk or touches production systems
- **Out of Scope** — explicit exclusions to prevent scope creep during execution
- **Reference Files** — key files in the codebase a model should read before starting

Use judgment. A small focused task may only need the mandatory sections. A multi-phase
refactor touching many systems warrants more.

---

## Step 4: Confirm and report

After writing the file, tell the user:
1. The full path of the saved plan (e.g., `~/.claude/plans/myapp-dashboard-refactor.md`)
2. How to load it in a future Claude Code session:
   - At the start of a new session: `Read ~/.claude/plans/{filename} and continue from the plan.`
   - Or ask Claude Code to find and load it: `Load my plan for {project} {feature}.`

---

## Referencing the plan in a future session

If the user asks you to reference, load, or continue from a plan and no file is
explicitly provided:

1. Run `ls ~/.claude/plans/` to list available plans
2. Identify the most likely match by project name and task slug
3. Use the Read tool to load that file
4. Confirm with the user before proceeding: "I found `{filename}` — is this the plan
   you want to use?"

If multiple plausible matches exist, list them and ask.

---

## Updating an existing plan

If a plan file for this project/task already exists and the user wants to update it
rather than create a new one:

- Preserve the filename unless the scope has changed significantly
- Add a `## Revision Notes` section at the bottom with a brief changelog entry
- Do not remove previous Key Decisions — append new ones

---

## Notes on quality

- Write requirements as assertions, not aspirations. "The API must return a response
  within 200ms under normal load" not "The API should be fast."
- Constraints are as important as requirements. An under-specified constraints section
  is the primary cause of execution drift.
- Acceptance criteria should be binary — either the criterion is met or it isn't.
- The plan file is a contract between the planner and the executor. Ambiguity in the
  plan becomes bugs or rework in execution.
