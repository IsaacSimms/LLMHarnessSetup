---
name: recap-thread
description: >
  Produce a concise, structured Markdown recap of the conversation that just happened, then save it
  as a .md file. Use when the user wants a record of work that is finished or winding down —
  phrases like "recap this", "summarize what we did", "write this up", "wrap up this thread",
  "document this fix", "log this", "save a summary", or "capture what we just figured out". Works for
  ANY kind of thread: break-fix and troubleshooting, ideation and brainstorming, research, or
  planning. IMPORTANT boundary with the thread-handoff skill: use recap-thread when the work is DONE
  and the user wants a durable record of what happened (backward-looking). Use thread-handoff instead
  when the user is CONTINUING the same work in a fresh session and needs to carry context forward
  (forward-looking). When a "summary" request is ambiguous: if the user is staying put and wants a
  keepsake or knowledge-base entry, choose recap-thread.
metadata:
  short-description: "Save a backward-looking thread recap"
---

# Recap Thread

Your job is to produce a **lightweight, structured recap** of the conversation that just happened and
save it as a `.md` file. This is a *backward-looking record* — it captures what the thread was about
and where it ended up, so the user has a durable artifact to file, search, or share later.

This is deliberately lighter than a full handoff document. It is not meant to let a fresh session
resume the work; it is meant to be a clean record of what happened. If the user actually wants to
continue the work elsewhere, that is the `thread-handoff` skill's job, not this one.

The recap must work for **any** kind of thread — a break-fix, an ideation session, a research dive, a
planning discussion. Use the single structure below for all of them; the section *names* are generic
on purpose so they hold any content. Sections that genuinely have nothing in them are omitted.

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

- **Lighter than a handoff. Density, not length.** There is no word cap, but every sentence earns its
  place. Length should scale with how substantial the thread was — a quick fix gets a few lines, a
  sprawling design session gets more.
- **Concrete over abstract.** Name the actual files, commands, error codes, hostnames, and decisions.
  "Fixed the boot issue" is weak. "Cleared stale PXE cert rows in the SQL DB, deleted RemoteInstall,
  error 0x80092002 resolved" is right.
- **No padding.** Skip "it may be worth noting", "in conclusion", and similar filler.
- **Summarize the work, not the chit-chat.** Capture substance and outcomes, not the back-and-forth.
- **Flag inferences.** If something is inferred rather than stated, mark it *(inferred)*.
- **Preserve the user's terminology.** If they call it a "task sequence", call it that.

---

## Where to Save

The recap is always written to a `.md` file. The filename is the same everywhere:

**`YYYY-MM-DD-topic-slug.md`** — today's date, then a kebab-case slug of ~3–6 words from the topic
(e.g. `2026-06-10-pxe-cert-corruption.md`). If a file with that name already exists in the target
directory, append `-2`, `-3`, etc. so nothing is overwritten.

Save to `~/.grok/summaries/`. Create the directory if it doesn't exist. After saving, tell the user
the full path.

If the session is sandboxed and only supports download-style output, save to the session outputs path
and present the file for download instead.

---

## After Saving

Keep it brief. Confirm where the file was saved (or present it for download), and optionally show the
TL;DR inline so the user sees the gist without opening the file. Do not paste the entire document back
into the chat — the file is the deliverable.

---

## After saving the recap, sweep context.md and Claude.md

Sweep the repo to see if you are in a directory with a `context.md` and/or `Claude.md` file. If so, analyze them and determine if any updates to those files are warranted based on the recap. If updates are needed, discuss with the user what changes should be made. If not, leave them as-is.
