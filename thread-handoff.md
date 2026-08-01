---
name: thread-handoff
description: >
  Generates a structured handoff document and saves it under docs/handoffs/ so a completely fresh
  CLI session can pick up exactly where this one left off. Covers two handoff modes:
  (1) implementation-to-implementation: resuming in-progress work in a new thread, and
  (2) ideation-to-implementation: handing a finalized design to an agent that pressure-tests then
  builds. Use when the user says "summarize this thread for a new session", "I'm starting a new
  thread", "context is getting long", "reset incoming", "handoff", "thread summary for a new chat",
  "write the handoff doc", or anything implying the conversation is ending and work continues
  elsewhere. Trigger proactively if the user is about to start fresh or wants to preserve progress.
  Use recap-thread instead when work is DONE and they only want a backward-looking record.
metadata:
  short-description: "Forward-looking handoff under docs/handoffs"
---

# Thread Handoff Skill

Produce a **Thread Handoff Document** — dense, structured — so a completely fresh session (zero
memory of this conversation) can act without redundant questions. **Always write the file** under
**`docs/handoffs/`**. Audience: **CLI coding agents** (Grok Build, Claude Code, peers) and the human.

The file is the source of truth. Do **not** paste the full document into chat.

---

## Routing (which skill)

| Situation | Skill | Location |
|-----------|--------|----------|
| Work **done**; durable record of what happened | `recap-thread` | `docs/recaps/` |
| Work **continues** in a fresh session | `thread-handoff` | `docs/handoffs/` |
| Decisions **settled**; implementable contract | `plan-as-artifact` | `docs/artifacts/` |
| Many open decisions / multi-session fog | `wayfinder` | `docs/plans/<map-slug>/` |
| Verify implement slice (build/tests/depth) | `check-work` | chat; optional `docs/checks/` |

A handoff may **link** a plan under `docs/artifacts/`; it does not replace the plan. Prefer
`plan-as-artifact` when the deliverable is a multi-phase execution contract rather than session
transfer.

---

## Resolve docs output directory

Shared by any skill that writes under `docs/`. This skill's subfolder is **`handoffs`**.

1. Start at the agent **cwd**.
2. If **`docs/` exists in cwd** → write under `{cwd}/docs/handoffs/`. Create `handoffs` if missing.
   **Do not ask.**
3. If no `docs/` in cwd → walk **up** looking for a **`docs/`** directory and/or a **git root**
   (`.git`).
4. If either is found → **ask the user** where to place the file. Offer defaults:
   - Nearest parent `docs/` → `{that-docs}/handoffs/`
   - Git root without `docs/` → create `{git-root}/docs/handoffs/`
   - If both exist and differ, list both; recommend nearest `docs/`
5. If walk-up finds **nothing** → **ask**, default: create `{cwd}/docs/handoffs/`.

Create intermediate directories as needed. Do **not** use home-dir folders as the primary write target.

---

## Step 1: Determine Handoff Mode

Before writing, classify the thread:

**Mode A — Implementation Handoff**
Active implementation work (code, scripts, config, debugging). The new thread resumes in place.

**Mode B — Ideation Handoff**
Designing / speccing / deciding — not building. The new thread pressure-tests the design then
implements.

State the mode explicitly at the top of the document. If the thread had both phases, use Mode A but
include an "Original Design" section.

---

## Step 2: Resolve filename and write the file

Filename: **`YYYY-MM-DD-topic-slug.md`** (today's date + kebab-case slug, ~3–6 words). If the name
exists, append `-2`, `-3`, etc.

Write to `{resolved-docs}/handoffs/{filename}`.

---

## Step 3: Handoff document body

### [MODE LABEL] — Thread Handoff Document

State at the top:

> **Handoff Mode: [Implementation | Ideation → Implementation]**
> **Receiving agent job: [Resume and continue | Pressure-test this design, then implement]**

---

### 1. Thread Purpose (2–4 sentences)
What this conversation was trying to accomplish. For ideation handoffs, what is fully decided and
ready for implementation.

---

### 2. Stack & Environment
*(Only what was discussed or confirmed)*
- Languages, frameworks, runtimes
- Tools, CLIs in use
- Platform/OS context
- Infrastructure or deployment context

---

### 3A. [Mode A only] What Was Accomplished
Ordered list of completed work. Specific over vague: name files, commands, outcomes.

### 3B. [Mode B only] Full Specification
Complete finalized design — architecture, data models, contracts, behavior rules, edge cases,
constraints, conventions. Enough fidelity that implementation can begin without re-asking design
questions.

---

### 4A. [Mode A only] Current State
Where things stand: what exists, partial, broken. The "you are here" marker.

### 4B. [Mode B only] What Is NOT Yet Decided
Open questions left for pressure-testing. If none: "All load-bearing decisions are made. No open
design questions remain."

---

### 5. Key Decisions & Rationale
Non-obvious decisions and *why*. The new thread must not re-litigate these.

| Decision | Rationale |
|----------|-----------|
| ... | ... |

---

### 6. [Mode A only] Blockers & Open Questions
Unresolved issues: what it is, what was tried, next step.

---

### 7. Next Steps (Ordered)
Numbered. First item = exactly what the new thread starts on.

For Mode B, step 1 is always: pressure-test the spec in §3B before writing any code.

---

### 8. Must-Knows for the New Thread
Constraints, user non-negotiables, pitfalls already hit, expected tone/framing.

---

### 9. Relevant Artifacts
File paths, plans under `docs/artifacts/`, configs, partial drafts — name, role, state.

---

## Tone & Quality Rules

- **Dense, not verbose.** Every sentence earns its place.
- **Concrete over abstract.** Name files, commands, errors, variables.
- **No hedging filler.**
- **Preserve exact terminology.**
- **Flag inference** with *(inferred)*.
- **Don't summarize the conversation.** Summarize work state (A) or design decisions (B).

---

## Step 4: Confirm and short opener

After writing the file:

1. Report the **full path**
2. Give a **short** paste-ready opener that points at the **file** (not the full body)

**Mode A:**

> Read `docs/handoffs/{filename}` and continue from that handoff. Confirm you have context and flag
> anything unclear before continuing.

**Mode B:**

> Read `docs/handoffs/{filename}`. Pressure-test the design first — challenge assumptions, surface
> gaps, identify contradictions — then implement once aligned. Do not start coding until the spec
> holds up.

Do **not** dump the full handoff into chat.

---

## Loading a handoff later

If the user asks to load a handoff without a path: search `docs/handoffs/` (resolve project as in
write). Confirm the match. There is no primary home-dir legacy location for handoffs (they were
chat-only before); if the user points at an old paste or other path, use that.
