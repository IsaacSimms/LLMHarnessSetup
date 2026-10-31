---
name: check-work
description: >
  Verify completed or in-progress work by delegating to the verifier-check subagent.
  Claimed behavior must be present, and the project's build and tests must pass.
  Use when the user says "check work", "verify changes", "self-verify", "/check-work",
  "/check", "/verify", or "/self-verify", or when offering verification after a non-trivial
  implement slice. Optional: save the report under docs/checks/ when the user asks to save
  a check-work doc; force-save there after three failed fix loops.
metadata:
  short-description: "Verify work via verifier-check"
---

# /check-work — Self-Verification

Verify work by delegating to the **verifier-check** subagent. That subagent reports.
This skill fixes real failures, re-verifies with a fresh `verifier-check` pass, and
stops cleanly. If `verifier-check` is not installed, say so and stop. Do not run the
pass in this session.

Audience: **CLI coding agents** (Grok Build, Claude Code, peers).

---

## Routing (sibling skills)

| Situation | Skill | Location |
|-----------|--------|----------|
| Work **done**; durable record | `recap-thread` | `docs/recaps/` |
| Work **continues** in a new session | `thread-handoff` | `docs/handoffs/` |
| Decisions **settled**; execution contract | `plan-as-artifact` | `docs/artifacts/` |
| Multi-session decision map | `wayfinder` | `docs/plans/<map-slug>/` |
| **Verify** this session's work | `check-work` | chat; optional `docs/checks/` |
| Stand up the project docs tree | `create-docs-folder` | `docs/` (`artifacts`, `recaps`, `handoffs`, `checks`, `plans`, `research`, `misc`) |
| Ambitious structural redesign review | `code-review` | (opt-in, separate) |

Standing up the whole tree is `create-docs-folder`. This skill still resolves its own output directory and creates only its own leaf when it writes.

check-work is a **gate**, not a redesign session. Deep code-judo belongs in `code-review`.

---

## When to run

| Mode | Behavior |
|------|----------|
| **Explicit** | User runs `/check-work`, `/check`, `/verify`, or asks to verify/check work. |
| **Offer** | After claiming a **non-trivial implement** slice is done (phase complete, “ready for review”), **offer** check-work once. Do **not** auto-run without the user accepting. |
| **Skip** | Pure design/grill/plan/Q&A with no code or operational deliverable. |

Optional focus: `/check-work [focus area]` — put that area in the task message. `verifier-check` centers on it.

---

## Mode detection

- **Same-turn**: User paired a task with check-work — **finish the task first**, then verify.
- **Standalone**: Only verify — go straight to verification.

---

## Resolve docs output directory (`docs/checks`)

Same algorithm as recap / handoff / plan-as-artifact. Subfolder: **`checks`**.

1. Start at agent **cwd**.
2. If **`docs/` exists in cwd** → use `{cwd}/docs/checks/`. Create `checks` if missing. **Do not ask.**
3. If no `docs/` in cwd → walk **up** for a **`docs/`** and/or **git root** (`.git`).
4. If either found → **ask** where to place the file; defaults:
   - Nearest parent `docs/` → `{that-docs}/checks/`
   - Git root without `docs/` → create `{git-root}/docs/checks/`
   - Both differ → list both; recommend nearest `docs/`
5. If walk-up finds nothing → **ask**; default `{cwd}/docs/checks/`.

Filename when saving: **`YYYY-MM-DD-topic-slug.md`**. If taken, append `-2`, `-3`, etc.

### When to write a check doc

| Trigger | Action |
|---------|--------|
| User asks to save (“save a check work doc”, “write the check report”, etc.) | Write report to `docs/checks/` after the verification pass (PASS or FAIL). |
| Default | **Chat only** — full structured report in chat; no file. |
| **3 fix loops exhausted** and still FAIL | **Force-write** the report to `docs/checks/`, **notify the user** with the path, and **stop**. |

---

## Orchestrator steps

1. **Narrate** that verification is starting (and any focus area).
2. Delegate to **verifier-check** as described under Delegation. Each recheck is a new invocation.
3. Read the result for exactly one `VERDICT: PASS` or `VERDICT: FAIL`.
4. **If PASS**: summarize evidence in chat; save to `docs/checks/` only if the user asked to save; stop.
5. **If FAIL**:
   - Narrate **explicitly**: what failed, evidence, what you will change.
   - Fix only the reported issues (minimal).
   - Narrate re-running verification (attempt N of 3).
   - Repeat from step 2.
   - **Max 3 fix→recheck loops.** After the third failed recheck: force-write `docs/checks/…`, notify user, stop. Do not thrash.

Always be explicit in chat while fix loops run — no silent retries.

---

## Delegation

Invoke the `verifier-check` subagent. Do not paste its procedure. Cursor loads that subagent from `~/.cursor/agents/verifier-check.md`.

The task message includes:

- The user asks for this session
- What was claimed done
- The focus area, when the user gave one
- The absolute repo path

`verifier-check` does not see this chat. It returns the report and does not edit the repo. This session makes any code changes.

If `verifier-check` is not installed, say so and stop.

Each recheck after a fix is a new `verifier-check` invocation.

---

## Chat report (always)

After each verification pass, show the user a concise version of the subagent report:

- What passed
- What was only claimed or is broken
- Build and test commands and their results
- Issues (if FAIL)
- `VERDICT: PASS` or `VERDICT: FAIL`
- Fix-loop status (`attempt k/3`) when applicable

Do not dump a wall of raw logs unless needed for evidence; keep exact failing command output.

---

## Saved check doc format (when writing `docs/checks/`)

```markdown
# Check-work: <topic>

**Date:** YYYY-MM-DD
**Verdict:** PASS | FAIL
**Focus:** <optional>
**Fix attempts:** <0–3>

## What passed
...

## Claimed or broken
...

## Build & Test Results
...

## Issues
...
```

---

## Notes on quality

- Prefer **fewer, deeper** tests as evidence — not coverage theater.
- PowerShell and C# are first-class; do not treat scripts as “untestable” by default.
- This skill does not replace TDD during build; it **verifies** after a slice.
- Explicit user waiver (“skip verify”, “don't check-work”) is honored and stated in the report.
