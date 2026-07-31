---
name: research
description: >
  Investigate a question against high-trust primary sources using a background subagent, and
  capture the findings as a committed Markdown file with per-claim citations, a confidence
  rating, and version stamps. Use this skill whenever the user wants a topic researched, docs
  or API behavior verified, a library or vendor evaluated, or reading legwork delegated —
  and always when resolving a wayfinder ticket of type `research`. Also use it proactively
  when you are about to answer a factual question about an external library, API, spec, or
  service from memory rather than from a source: if the answer matters and you have not read
  the source, research it instead of recalling it.
---

# Research

Spin up a **background subagent** to do the reading, so the calling session keeps its context
for the work that needs it. Context isolation is the entire point — if the findings come back
into the calling session as a file plus a summary rather than as fifty pages of documentation,
the skill worked.

## The division of labour

The subagent **reads and recommends**. The calling session **decides**.

The subagent finishes with the deepest context on the sources and the shallowest on the
problem — it never saw the destination, the constraints, or the decisions already made. A
recommendation from something that actually read the docs is valuable. A decision from
something that has not read the map is not.

So a research subagent never closes a ticket, never edits a plan, and never writes code. It
writes one file and returns a pointer.

## Modes

Determine the mode from the caller, not from the question.

| Mode | Trigger | Output location | Filename |
|---|---|---|---|
| Wayfinder | Resolving a `research` ticket | `docs/plans/<map-slug>/research/` | Ticket's slug: `002-graphql-subscription-limits.md` |
| Standalone | Everything else | `docs/research/` | Slug of the question: `graphql-subscription-limits.md` |

Both locations are **fixed**. Do not go looking for where the repo "already keeps such notes"
and do not pick somewhere sensible — that produces a different location every run, and
research notes scattered across a repo are unfindable, which defeats writing them down.
Create the directory if it does not exist.

Filenames derive from the ticket or the question. Never allocate a sequential number:
wayfinder fires several research subagents in parallel during charting, and each would list
the directory and take the same next number.

## Source tiers

Cite the tier alongside every claim. If a claim rests only on T3, say so — that is a finding
about the state of the evidence, not a defect in the research.

- **T1 — the thing itself.** Source code, the spec, the reference implementation, the API's
  own schema. This includes reading `node_modules`, vendored code, or a cloned repo, which
  frequently beats the documentation and is always more current than it.
- **T2 — first-party publication.** Official docs, changelogs, release notes, migration guides.
- **T3 — maintainer testimony.** Statements by maintainers in issues, PRs, RFCs, design docs.

**Not primary, do not cite as evidence:** blog posts, Stack Overflow, tutorials, courses,
aggregator sites, AI-generated summaries. They are useful for *finding* T1–T3 sources. Follow
them to the source that owns the claim, cite that, and drop the intermediary.

When a T3 source contradicts T1, T1 wins and the contradiction is itself worth recording.

## Stopping rule

Stop when **the question posed is answered**, or when **three consecutive sources add nothing
new**. Hard ceiling of ~20 source fetches as a backstop only, not as a target.

Do not stop on a turn or token budget — that halts mid-thought and returns half-findings that
look complete. Do not chase a source count — that rewards five shallow sources over two deep
ones, which is backwards.

If the ceiling is reached with the question unanswered, that is an `inconclusive` result. Say
so. Do not pad.

## Output format

One Markdown file. Nothing else.

```markdown
---
question: <the question, verbatim as posed>
researched: <YYYY-MM-DD>
confidence: confident | partial | inconclusive
versions:
  - <library/API/spec>: <version or revision checked>
---

# <Question as a heading>

## Answer

<Direct answer to the question posed. Two or three sentences. If inconclusive, say what is
not knowable from available sources and stop — do not substitute reasoning for evidence.>

## Findings

- <Claim.> — [T1] <source name>(<url or path>)
- <Claim.> — [T2] <source name>(<url or path>)

## Recommendation

<What the subagent would do, and why. Explicitly a recommendation, not a decision. Name the
tradeoff it turns on so the calling session can weigh it against context the subagent lacks.>

## Ruled out

<Options considered and eliminated, with the reason. Also: sources checked that had nothing —
this is what stops the next session from re-reading them.>

## Open questions

<What this research surfaced that it could not resolve.>
```

`## Confidence` is not optional and `inconclusive` is a legitimate, valuable result. "The docs
do not specify this" is a finding — it tells the calling session to design for the ambiguity
rather than assume a behavior. A template with no slot for it pressures a subagent into
manufacturing certainty to fill the space.

`versions:` is what makes a finding falsifiable six months from now. A committed research file
with no version stamp cannot be checked for staleness and will quietly mislead.

## Subagent brief

The calling session passes this. Fill every slot before spawning.

```
Research this question and write your findings to <full output path>.

QUESTION: <the question, stated precisely and narrowly>

CONTEXT: <why this is being asked — enough to judge relevance, not the whole problem>

Use only primary sources, tiered T1 (source code, specs, schemas — including node_modules
and vendored code), T2 (official docs, changelogs), T3 (maintainer statements in issues,
PRs, RFCs). Blog posts and Stack Overflow may be used to FIND primary sources but never
cited as evidence. Tag every claim with its tier.

Stop when the question is answered, or when three consecutive sources add nothing new.
Hard ceiling ~20 fetches. If you hit the ceiling unresolved, report confidence:
inconclusive — do not pad.

Write exactly one file, in the format specified by the `research` skill. Do not edit any
other file, do not write code, do not close any ticket. Return the file path and a
three-line summary.
```

## When the calling session gets the result

1. Read the file. Do not re-read its sources — that discards the context isolation you paid for.
2. Weigh the recommendation against what the subagent could not see.
3. **Wayfinder mode:** write the ticket's `## Answer` — the decision, plus reasoning, plus a
   link to the research file. Close the ticket. The research file is an asset the ticket links,
   never content pasted into it.
4. **Standalone mode:** answer the user, citing the file path so the findings are findable later.

## No subagents available

If the runtime cannot spawn a background agent, run the research **inline** in the current
session and state plainly in the output file and to the user that context isolation was lost.

Silent degradation is worse than none: an inline research pass burns the calling session's
context on documentation, which is the exact cost this skill exists to avoid. When running
inline, narrow the question first — a question that would have been fine for a subagent is
often too broad to absorb inline.
