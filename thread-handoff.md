---
name: thread-handoff
description: >
  Generates a structured handoff document summarizing the current thread so a completely fresh
  conversation can pick up exactly where this one left off — with no context loss. Use this skill
  whenever the user says things like "summarize this thread for a new session", "I'm starting a new
  thread", "context is getting long", "reset incoming", "handoff", "thread summary for a new chat",
  or anything implying the current conversation is ending and work needs to continue elsewhere.
  Always trigger this skill proactively if the user says they're about to start fresh or the
  conversation has been going long and they want to preserve progress.
---

# Thread Handoff Skill

Your job is to produce a **Thread Handoff Document** — a dense, structured summary that allows a
completely fresh Claude session (with zero memory of this conversation) to pick up exactly where
this thread left off.

Write as if briefing a highly competent engineer who has never spoken to the user before. Assume
nothing carries over. Every relevant decision, constraint, finding, and next step must be explicit.

---

## Output Structure

Produce the handoff in this exact order. Omit sections that are genuinely empty (e.g., no blockers
exist). Do not add sections not listed here unless a project-specific section is obviously needed.

---

### 1. Thread Purpose (2–4 sentences)
What was this conversation trying to accomplish? State the goal and scope plainly.

### 2. Stack & Environment
Bullet list. Include only what was actually discussed or confirmed. Cover:
- Languages, frameworks, runtimes
- Tools, IDEs, CLIs in use
- Relevant platform/OS context (e.g., Windows, SCCM env, Azure tenant)
- Any infrastructure or deployment context that matters

### 3. What Was Accomplished
Ordered list of completed work — decisions made, implementations done, problems solved.
Be specific. "Wrote the deployment script" is weak. "Wrote `Deploy-WSLKernel.ps1` targeting SYSTEM
account with fallback logging to `C:\Windows\Temp\`" is correct.

### 4. Current State
Describe exactly where things stand right now. What exists, what is partially done, what is broken.
This is the "you are here" marker for the new thread.

### 5. Key Decisions & Rationale
Table or bullet list of non-obvious decisions made during this thread and *why* they were made.
The new thread needs to know not just what was chosen, but why — so it doesn't re-litigate settled
questions.

| Decision | Rationale |
|----------|-----------|
| ... | ... |

### 6. Blockers & Open Questions
Unresolved issues, things that failed, questions that were raised but not answered.
Each item should state: what it is, what was tried (if anything), and what the next step is.

### 7. Next Steps (Ordered)
Numbered list. The first item should be exactly what the new thread should start working on.
Be specific enough that the new session can act immediately without asking clarifying questions.

### 8. Must-Knows for the New Thread
Critical context that doesn't fit neatly above but would cause the new session to make mistakes
or ask redundant questions without it. Examples:
- Constraints that were discovered mid-thread ("SYSTEM account can't access network shares")
- Conventions the user has established ("use guard clauses, no nested ifs")
- Preferences or non-negotiables ("always use ConfigMgr baseline detection, not registry keys")
- Pitfalls already hit ("tried X, it breaks because Y")

### 9. Relevant Artifacts
List any code, scripts, configs, or documents produced or referenced in this thread:
- File name / location
- What it does
- Its current state (complete, draft, broken, needs review)

If snippets are short and critical, include them inline. For long artifacts, describe them and note
where they live.

---

## Tone & Quality Rules

- **Dense, not verbose.** Every sentence earns its place.
- **Concrete over abstract.** Name the files, commands, error messages, variable names.
- **No hedging filler.** Do not write "it might be worth noting" or "you may want to consider".
- **Preserve exact terminology.** If the user calls something a "detection method", call it that.
- **Flag confidence.** If something is inferred rather than explicitly confirmed, note it with *(inferred)*.
- **Don't summarize the conversation.** Summarize the *work and its state*.

---

## Trigger Phrase to Pass to the New Thread

End the handoff document with a ready-to-paste opening message for the new thread:

> **Paste this into your new thread to resume:**
>
> "Picking up from a previous session. Here's the handoff: [paste the full document above]
> Please confirm you have context and tell me if anything is unclear before we continue."
