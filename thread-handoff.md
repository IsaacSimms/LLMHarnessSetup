---
name: thread-handoff
description: >
  Generates a structured handoff document so a completely fresh session can pick up exactly
  where this one left off — with no context loss. Covers two handoff modes:
  (1) implementation-to-implementation: resuming in-progress work in a new thread, and
  (2) ideation-to-implementation: handing a finalized design to an agent whose job is to
  pressure-test then build it.
  Use whenever the user says things like "summarize this thread for a new session", "I'm starting
  a new thread", "context is getting long", "reset incoming", "handoff", "thread summary for a
  new chat", "write the handoff doc", or anything implying the current conversation is ending and
  work needs to continue elsewhere. Always trigger proactively if the user says they're about to
  start fresh or the conversation has been going long and they want to preserve progress.
---

# Thread Handoff Skill

Your job is to produce a **Thread Handoff Document** — a dense, structured summary that allows a
completely fresh Claude session (with zero memory of this conversation) to immediately understand
context and act without asking redundant questions.

---

## Step 1: Determine Handoff Mode

Before writing, classify the thread into one of two modes:

**Mode A — Implementation Handoff**
The thread was doing active implementation work (writing code, building scripts, configuring
systems, debugging). Work exists in some state. The new thread resumes where this one left off.

**Mode B — Ideation Handoff**
The thread was designing, speccing, or deciding — not building. The output is a fully-specified
design. The new thread's job is to pressure-test the design then implement it.

State the mode explicitly at the top of the document. If the thread had both phases (started
ideating, then began implementing), use Mode A but include an "Original Design" section.

---

## Step 2: Write the Handoff Document

---

### [MODE LABEL] — Thread Handoff Document

State at the top:
> **Handoff Mode: [Implementation | Ideation → Implementation]**
> **Receiving agent job: [Resume and continue | Pressure-test this design, then implement]**

---

### 1. Thread Purpose (2–4 sentences)
What was this conversation trying to accomplish? State the goal and scope plainly.
For ideation handoffs, state what is now fully decided and ready for implementation.

---

### 2. Stack & Environment
*(Include only what was actually discussed or confirmed)*
- Languages, frameworks, runtimes
- Tools, IDEs, CLIs in use
- Platform/OS context (e.g., Windows, SCCM env, Azure tenant)
- Infrastructure or deployment context

---

### 3A. [Mode A only] What Was Accomplished
Ordered list of completed work — decisions made, implementations done, problems solved.
Specific over vague: "Wrote `Deploy-WSLKernel.ps1` targeting SYSTEM account with fallback
logging to `C:\Windows\Temp\`" not "wrote a deployment script."

### 3B. [Mode B only] Full Specification
The complete, finalized design as decided in this thread. This is the authoritative spec the
implementation agent will build from. Include every load-bearing decision:
- Architecture / component breakdown
- Data models, schemas, contracts
- Behavior rules, edge cases, constraints
- Named conventions and taxonomy
- Anything that was explicitly finalized ("everything load-bearing is decided")

Do not summarize — reproduce the design with enough fidelity that implementation can begin
without re-asking any design question.

---

### 4A. [Mode A only] Current State
Exactly where things stand. What exists, what is partially done, what is broken.
This is the "you are here" marker.

### 4B. [Mode B only] What Is NOT Yet Decided
List any open questions, deferred choices, or things explicitly left for the implementation
agent to resolve during pressure-testing. If nothing is open, state that explicitly:
"All load-bearing decisions are made. No open design questions remain."

---

### 5. Key Decisions & Rationale
Non-obvious decisions made in this thread and *why*. The new thread must not re-litigate these.

| Decision | Rationale |
|----------|-----------|
| ... | ... |

---

### 6. [Mode A only] Blockers & Open Questions
Unresolved issues, things that failed, questions raised but not answered.
Each item: what it is, what was tried, what the next step is.

---

### 7. Next Steps (Ordered)
Numbered list. First item = exactly what the new thread starts on.
Specific enough to act immediately.

For Mode B, step 1 is always: "Pressure-test the spec in §3B before writing any code.
Challenge assumptions, surface contradictions, identify missing edge cases."

---

### 8. Must-Knows for the New Thread
Critical context that doesn't fit above but would cause mistakes or redundant questions:
- Constraints discovered mid-thread
- User conventions and non-negotiables
- Pitfalls already hit ("tried X, breaks because Y")
- Tone/framing the user expects from the new agent

---

### 9. Relevant Artifacts
*(Mode A primarily; Mode B if any reference docs or partial drafts exist)*
- File name / location
- What it does
- Current state (complete, draft, broken, needs review)

For short critical snippets, include inline. For long artifacts, describe and note location.

---

## Tone & Quality Rules

- **Dense, not verbose.** Every sentence earns its place.
- **Concrete over abstract.** Name the files, commands, error messages, variable names.
- **No hedging filler.** Cut "it might be worth noting" and "you may want to consider."
- **Preserve exact terminology.** If the user calls it a "detection method," call it that.
- **Flag inference.** If something is inferred rather than explicitly confirmed: *(inferred)*
- **Don't summarize the conversation.** Summarize the *work and its state* (Mode A) or
  the *design and its decisions* (Mode B).

---

## Closing: Ready-to-Paste Opener

End the document with the appropriate paste-in opener for the new thread:

**Mode A:**
> **Paste into new thread:**
> "Picking up from a previous session. Here's the handoff: [paste document]
> Confirm you have context and flag anything unclear before we continue."

**Mode B:**
> **Paste into new thread:**
> "Here's a fully-specified feature design from an ideation session. Your job is to
> pressure-test this spec first — challenge assumptions, surface gaps, identify contradictions —
> then implement it once we've aligned. Here's the spec: [paste document]
> Start by grilling the design before writing any code."
