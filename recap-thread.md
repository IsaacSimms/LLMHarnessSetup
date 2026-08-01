b---
name: recap-thread
description: >
  Produce a concise, structured Markdown recap of the conversation that just happened, then save it
  under docs/recaps/ in the project. Use when the user wants a record of work that is finished or
  winding down — phrases like "recap this", "summarize what we did", "write this up", "wrap up this
  thread", "document this fix", "log this", "save a summary", or "capture what we just figured out".
  Works for ANY kind of thread: break-fix, ideation, research, or planning. IMPORTANT boundary with
  thread-handoff: use recap-thread when the work is DONE and the user wants a durable record
  (backward-looking). Use thread-handoff when the user is CONTINUING the same work in a fresh session
  (forward-looking). When "summary" is ambiguous: if the user is staying put and wants a keepsake,
  choose recap-thread.
metadata:
  short-description: "Save a backward-looking thread recap under docs/recaps"
---

# Recap Thread

Produce a **lightweight, structured recap** of the conversation and save it as a `.md` file under
**`docs/recaps/`**. This is a *backward-looking record* — what the thread was about and where it
ended — for filing, search, or share later.

It is deliberately lighter than a handoff. It is **not** meant to let a fresh session resume work;
that is `thread-handoff`. Audience: humans and **CLI coding agents** (Grok Build, Claude Code, peers).

---

## Routing (which skill)

| Situation | Skill | Location |
|-----------|--------|----------|
| Work **done**; durable record of what happened | `recap-thread` | `docs/recaps/` |
| Work **continues** in a fresh session | `thread-handoff` | `docs/handoffs/` |
| Decisions **settled**; implementable contract | `plan-as-artifact` | `docs/artifacts/` |
| Many open decisions / multi-session fog | `wayfinder` | `docs/plans/<map-slug>/` |

---

## Resolve docs output directory

Shared by any skill that writes under `docs/`. This skill's subfolder is **`recaps`**.

1. Start at the agent **cwd**.
2. If **`docs/` exists in cwd** → write under `{cwd}/docs/recaps/`. Create `recaps` if missing.
   **Do not ask.**
3. If no `docs/` in cwd → walk **up** looking for a **`docs/`** directory and/or a **git root**
   (`.git`).
4. If either is found → **ask the user** where to place the file. Offer defaults:
   - Nearest parent `docs/` → `{that-docs}/recaps/`
   - Git root without `docs/` → create `{git-root}/docs/recaps/`
   - If both exist and differ, list both; recommend nearest `docs/`
5. If walk-up finds **nothing** → **ask**, default: create `{cwd}/docs/recaps/`.

Create intermediate directories as needed. Do **not** use home-dir folders as the primary write target.

---

## Output Structure

Produce the document in exactly this order.

```
# <Topic as a short title>

**Date:** YYYY-MM-DD
**Type:** <fix | ideation | research | planning | ...>
**Environment / Systems:** <only if relevant>
**People / Teams:** <only if any were involved>
**Ticket #:** <only if one came up>

## TL;DR
<1–3 sentences. The whole thread in a nutshell. Always present, always first.>

## Context & Goal
<What prompted this and what we were trying to accomplish.>

## Key Points Explored
<The substance. For a fix: symptoms, investigation, root cause. For ideation: the ideas and
options weighed. For research: what was found. Bullets are fine.>

## Decisions & Outcomes
<What was concluded, chosen, resolved, or shipped. For a fix, the remediation + how it was verified
goes here.>

## Open Questions / Next Steps
<Anything unresolved, deferred, or worth doing next. Omit if truly nothing is open.>

## Artifacts
<Scripts, configs, commands, file paths, links, or docs produced or referenced. Name them and note
their state. Omit if none.>
```

### Rules for the metadata block

- **Date** is always today's date.
- Every other field appears **only if it has real content.** Do not emit `Environment / Systems: N/A`
  — just drop the line.
- **Type** is one lowercase word that best fits the thread. Invent one if none of the examples fit.

### Rules for the body

- **TL;DR is mandatory and always first** after the metadata block.
- Use the section names above. You may *omit* an empty section, but do not rename or reorder the ones
  you keep, and do not invent new top-level sections unless the thread genuinely demands one.
- For a break-fix thread, the 5 W's land naturally: *who/where* in the metadata, *what/why* in Context
  & Key Points, the fix in Decisions & Outcomes. Don't force literal "Who:/What:" labels.

---

## Density & Quality Rules

- **Lighter than a handoff. Density, not length.** Length scales with how substantial the thread was.
- **Concrete over abstract.** Name actual files, commands, error codes, hostnames, decisions.
- **No padding.** Skip "it may be worth noting", "in conclusion", and similar filler.
- **Summarize the work, not the chit-chat.**
- **Flag inferences.** Mark inferred facts *(inferred)*.
- **Preserve the user's terminology.**

---

## Where to Save

Filename: **`YYYY-MM-DD-topic-slug.md`** — today's date, then a kebab-case slug of ~3–6 words
(e.g. `2026-06-10-pxe-cert-corruption.md`). If that name exists in the target directory, append
`-2`, `-3`, etc. so nothing is overwritten.

Write to `{resolved-docs}/recaps/{filename}` using **Resolve docs output directory** above.

After saving, tell the user the full path.

**Legacy:** Older recaps may live under `~/.grok/summaries/`. Do **not** auto-migrate. When the
user asks to find a recap and nothing is under `docs/recaps/`, search that legacy path and confirm
before treating a hit as the answer.

If the session is sandboxed and only supports download-style output, save to the session outputs
path and present the file for download instead.

---

## After Saving

Keep it brief. Confirm where the file was saved (or present it for download), and optionally show
the TL;DR inline. Do **not** paste the entire document back into the chat — the file is the
deliverable.

---

## Context file sweep (after save)

After saving, check the resolved project root for any of:

`CONTEXT.md`, `Context.md`, `context.md`, `Claude.md`, `GROK.md`, `AGENTS.md`

If present, decide whether the recap implies durable project knowledge that belongs in those files.
If updates seem warranted, **discuss with the user before editing** — never silent rewrite. If not
warranted, leave them as-is.
