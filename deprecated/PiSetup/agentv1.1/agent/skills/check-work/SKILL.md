---
name: check-work
description: >
  Verify completed or in-progress work: run stack-aware builds/tests, review the session
  against the request (and plan AC when present), and run a short depth/UL design pass.
  Use when the user says "check work", "verify changes", "self-verify", "/check-work",
  "/check", "/verify", or "/self-verify", or when offering verification after a non-trivial
  implement slice. Optional: save the report under docs/checks/ when the user asks to save
  a check-work doc; force-save there after three failed fix loops.
metadata:
  short-description: "Verify work: tests, plan AC, depth/UL"
---

# /check-work — Self-Verification

Verify work by running a structured verification pass (prefer a **fresh verifier subagent**
when available; otherwise a distinct second-pass with the same prompt). Fix real failures,
re-verify, and stop cleanly.

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
| Ambitious structural redesign review | `code-review` | (opt-in, separate) |

check-work is a **gate**, not a redesign session. Deep code-judo belongs in `code-review`.

---

## When to run

| Mode | Behavior |
|------|----------|
| **Explicit** | User runs `/check-work`, `/check`, `/verify`, or asks to verify/check work. |
| **Offer** | After claiming a **non-trivial implement** slice is done (phase complete, “ready for review”), **offer** check-work once. Do **not** auto-run without the user accepting. |
| **Skip** | Pure design/grill/plan/Q&A with no code or operational deliverable. |

Optional focus: `/check-work [focus area]` — verifier prioritizes that area.

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
2. Run the **VERIFIER PROMPT** (below) via a fresh subagent when available, or a second-pass in-process with the same structure. Prefer isolation so the verifier re-checks claims instead of rubber-stamping.
3. Read the result for `VERDICT: PASS` or `VERDICT: FAIL`.
4. **If PASS**: summarize evidence in chat; save to `docs/checks/` only if the user asked to save; stop.
5. **If FAIL**:
   - Narrate **explicitly**: what failed, evidence, what you will change.
   - Fix only the reported issues (minimal).
   - Narrate re-running verification (attempt N of 3).
   - Repeat from step 2.
   - **Max 3 fix→recheck loops.** After the third failed recheck: force-write `docs/checks/…`, notify user, stop. Do not thrash.

Always be explicit in chat while fix loops run — no silent retries.

---

## VERIFIER PROMPT

Copy verbatim into the verifier. Append **Additional Focus** if the user gave a focus area.

```
You are an expert verifier. Determine whether work in this session correctly and
completely addresses the user's requests. Do not trust the parent agent's claims —
re-verify with tools and the environment.

=== SCOPE ===

- If Additional Focus is present: center the verdict on that area; still note blockers elsewhere.
- Otherwise: verify all work done in this session that the user asked for.

=== PHASE A: TRACE REVIEW (always) ===

1. UNDERSTAND THE REQUEST
   Restate user asks across the session as a concrete checklist (features, fixes,
   configs, research answers, git/PR tasks, etc.).

2. RECONSTRUCT WHAT HAPPENED
   What was attempted, what succeeded, what was skipped, deferred, or only claimed.

3. VERIFY CURRENT STATE
   Inspect the environment: read changed files, confirm commands' effects, confirm
   resources exist. Do not accept conversation claims as proof.

=== PHASE B: CODE / BUILD / TEST (when code or scripts were touched) ===

Skip only for pure non-code Q&A with no file or operational deliverable.

4. COLLECT DIFF / SCOPE
   Use git diff (staged, unstaged, recent commits) and/or the files known changed.
   Read surrounding context.

5. BUILD AND TEST (stack-aware)
   Discover and run the real project commands. Prefer repo docs (README, AGENTS.md,
   CLAUDE.md, GROK.md, package scripts) when they define the canonical command.

   Detection defaults (use when docs don't specify):
   - C# / .NET: *.sln or *.csproj → `dotnet build`, then `dotnet test`
     (narrow with filter if focus area names a test project/class).
   - PowerShell: *Tests.ps1 / Pester layout → Invoke-Pester (or project script).
   - Node: package.json test/build scripts when present.
   - Python: pytest / project script when present.
   - Other: follow README; if nothing exists, do not invent a green suite.

   Rules:
   - Broken build → automatic FAIL.
   - Failing tests → automatic FAIL.
   - No test runner found → do not silent PASS. Report what was checked (e.g. build
     only, static read) and FAIL if behavior claims are unproven, or state clearly
     that verification is partial and why.

6. DESIGN AND RUN EXTRA CHECKS (as needed)
   Targeted smoke tests, CLI invokes, curl, temp fixtures for scripts — only when
   they add evidence. Clean up temp artifacts when practical.

7. TEST QUALITY (if tests were added/changed)
   - Behavior through public interface / exported surface, not private guts.
   - Not over-mocked past real seams.
   - Names describe WHAT, not HOW.
   - RED should have been for missing behavior, not plumbing (if history shows otherwise, note it).

=== PHASE C: DEPTH / UL (short pass; when code changed) ===

Use the user's Ubiquitous Language if present (Module, Interface, Seam, Adapter,
Depth). FAIL only on **clear** problems; otherwise suggestion.

FAIL-worthy examples:
- New shallow pass-through module (deletion test fails — complexity only moves to callers).
- New seam/abstraction with only one adapter and no real variation.
- Tests that mock past the seam under test or target private implementation.
- Clear violation of "edit/refactor existing code first" — unjustified new files/types
  when an existing module should have been extended.
- Clear violation of stated project rules in CLAUDE.md / GROK.md / AGENTS.md.

Suggestion-only (do not FAIL alone):
- "Could be deeper" without a concrete structural defect.
- Style nits, alternate designs, ambitious code-judo (point user at code-review skill).

=== PLAN AWARENESS ===

If the session used or named a plan under docs/artifacts/ (or equivalent):
- Include plan Requirements and phase acceptance criteria in the checklist.
- Incomplete Progress items without user acceptance of partial work → FAIL or
  explicit partial verdict with what's left.

=== VERDICT ===

End with exactly one of:
VERDICT: PASS
VERDICT: FAIL

FAIL: precise issues, evidence (command output, paths, lines), what must change.
PASS: what was run and what evidence supports success.

=== PRINCIPLES ===

- Outcomes over claims. Proxy signals (effort, partial tests) do not cover unmet checklist items.
- Do not invent issues to fill space.
- Policy rules in project instruction files are FAILs when violated; vague taste is not.
- Temporary verify-only files are allowed if cleaned up when practical.

=== OUTPUT FORMAT ===

## Checklist
## Action Trace
## Diff Summary / Code Scope (if Phase B)
## Evaluation
- Correctness / Adequacy / Excess / Edge cases
- Depth/UL (Phase C)
## Build & Test Results
commands + outcomes
## Issues
### Issue N -- Severity: bug|gap|regression|depth|suggestion
- File / evidence / fix suggestion
## Verdict line
VERDICT: PASS|FAIL
```

### Additional Focus (orchestrator appends when present)

```
## Additional Focus
<focus area text>
Pay special attention to these areas during verification.
```

---

## Chat report (always)

After each verification pass, show the user a concise version of:

- Checklist status
- Build & test commands + results
- Depth/UL findings (if any)
- Issues (if FAIL)
- `VERDICT: PASS|FAIL`
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

## Checklist
...

## Build & Test Results
...

## Depth / UL
...

## Issues
...

## Action Trace (summary)
...
```

---

## Notes on quality

- Prefer **fewer, deeper** tests as evidence — not coverage theater.
- PowerShell and C# are first-class; do not treat scripts as “untestable” by default.
- This skill does not replace TDD during build; it **verifies** after a slice.
- Explicit user waiver (“skip verify”, “don't check-work”) is honored and stated in the report.
