---
name: research
description: >
  Investigate a question against high-trust primary sources by delegating to the
  researcher subagent, and capture the findings as a Markdown file on disk with
  per-claim citations, a confidence rating, and version stamps. Use this skill whenever
  the user wants a topic researched, docs or API behavior verified, a library or vendor
  evaluated, or reading legwork delegated — and always when resolving a wayfinder ticket
  of type `research`. Also use it proactively when you are about to answer a factual
  question about an external library, API, spec, or service from memory rather than from
  a source: if the answer matters and you have not read the source, research it instead
  of recalling it.
---

# Research

Delegate the reading to the **researcher** subagent. It runs in the background, writes one
findings file, and returns the path plus a three-line summary, so this session keeps its
context for the decision. If the findings come back as that file plus the summary rather
than as fifty pages of documentation, the skill worked.

If `researcher` is not installed, say so and stop. Do not research in this session.

## The division of labour

`researcher` **reads and recommends**. This session **decides**.

`researcher` finishes with the deepest context on the sources and the shallowest on the
problem. It never saw the destination, the constraints, or the decisions already made. A
recommendation from something that actually read the docs is valuable. A decision from
something that has not read the map is not.

`researcher` never closes a ticket, never edits a plan, and never writes code. It writes
one file and returns a pointer. Do not create a git commit as part of this skill.

## Modes

Determine the mode from the caller, not from the question.

| Mode | Trigger | Output location | Filename |
|---|---|---|---|
| Wayfinder | Resolving a `research` ticket | `docs/artifacts/maps/<map-slug>/research/` | Ticket's slug: `002-graphql-subscription-limits.md` |
| Standalone | Everything else | `docs/research/` | Slug of the question: `graphql-subscription-limits.md` |

`docs/research/` is a leaf `create-docs-folder` scaffolds. Both locations are **fixed**. Do not go looking for where the repo "already keeps such notes"
and do not pick somewhere sensible — that produces a different location every run, and
research notes scattered across a repo are unfindable, which defeats writing them down.
Create the directory if it does not exist.

Filenames derive from the ticket or the question. Never allocate a sequential number:
wayfinder fires several `researcher` invocations in parallel during charting, and each would list
the directory and take the same next number.

## Audience

Default: the findings file is for the user. Tell `researcher` to keep it digestible: scannable,
short claims, no wall of text.

If the user says the research, a specific subagent run, or a specific file is for an agent
("this one is for an agent", "the next agent will read it", "I won't read it"), the audience is
an agent. Tell `researcher` the user will not read the file, so it may be as long, verbose, and
dense as the evidence needs. Completeness beats skimmability. This overrides the length guidance
in its output format (for example, the short Answer). Keep the format's sections, frontmatter,
tiers, and citations. The three-line return summary stays short either way.

The user may scope this to one invocation or the whole run. Apply it to exactly what they scoped.
With several `researcher` invocations in a run, set AUDIENCE per invocation.

## Delegation

Invoke `researcher` with the Agent tool (`subagent_type: researcher`). Do not paste its procedure.

Fill every slot before spawning. The output path comes from Modes. AUDIENCE comes from Audience.

```
Research this question and write your findings to <full output path>.

QUESTION: <the question, stated precisely and narrowly>

CONTEXT: <why this is being asked — enough to judge relevance, not the whole problem>

AUDIENCE: <user: keep the file digestible, not a wall of text | agent: the user will not read
this file, an agent will; be as long and detailed as the evidence needs>
```

Each wayfinder research ticket is its own `researcher` invocation, with its own output path.

If `researcher` is not installed, say so and stop.

## Conflict pass

When `researcher` returns, invoke `content-conflict-resolver` (`subagent_type:
content-conflict-resolver`) on every findings file from the run, passing their paths. It runs in
the foreground and finishes before the readability pass starts. Run it even when the run produced
one file. It applies to agent-audience files too.

## Readability pass

When the conflict pass returns, invoke `readability-verifier` (`subagent_type:
readability-verifier`) on the findings file path. It runs in the foreground. This is the last pass
over the file: the research is not complete, and no ticket is closed or answer given, until it
returns.

Skip this pass for any file whose audience is an agent (see Audience). That file is meant to be
long and dense, and the pass would undo it.

## When the calling session gets the result

1. Read the file. Do not re-read its sources — that discards the context isolation you paid for.
2. Weigh the recommendation against what `researcher` could not see.
3. **Wayfinder mode:** write the ticket's `## Answer` — the decision, plus reasoning, plus a
   link to the research file. Close the ticket. The research file is an asset the ticket links,
   never content pasted into it.
4. **Standalone mode:** answer the user, citing the file path so the findings are findable later.
