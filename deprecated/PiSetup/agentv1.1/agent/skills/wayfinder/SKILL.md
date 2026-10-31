---
name: wayfinder
description: >
  Plan work too large for one agent session as a markdown map of decision tickets under
  docs/plans/, then resolve them one at a time until the way to the destination is clear.
  Use this skill whenever a task is too big to hold in a single session, whenever the user
  says "chart this", "map this out", "this is a big one", "wayfinder", "work the map", or
  names a map by slug or path. Also use it proactively when the user describes a multi-week
  effort, a migration, a rewrite, a greenfield build, or any request where the sequence of
  decisions is not yet visible — even if they did not ask for a plan. If you catch yourself
  about to write a long linear TODO list for a large ambiguous effort, use this skill instead.
---

# Wayfinder

A loose idea has arrived, too big for one session and wrapped in fog: the way from here to
the **destination** is not visible yet. This skill charts that way as a **map** of markdown
files, then works its **decision tickets** — questions whose resolution is a decision, not
slices of a build — one at a time until the route is clear.

## Plan, don't do

Wayfinder is planning. Each ticket resolves a decision. The map is done when nothing is left
to decide.

The pull to just start coding is the signal you have reached the edge of the map, not
permission to cross it. Producing decisions, not deliverables, is the whole point: a ticket
is a *question*, and questions are cheap to reorder, invalidate, or delete when an upstream
decision changes. Half-written code is not.

Execution happens after the map closes, from the `work/` files the map emits (see
[Emit work items](#mode-c-emit-work-items)).

## Substrate

Everything is markdown, committed to the repo:

```
docs/plans/<map-slug>/
  map.md
  tickets/
    001-choose-auth-strategy.md
    002-pick-data-access-seam.md
  work/
    001-build-repo-and-migrations.md
```

Rules:

- **Numbering** — zero-padded, sequential, allocated per directory. Before creating a ticket
  or work item, list the directory and take the next number. Numbers are never reused, even
  after deletion.
- **Paths never change.** Status lives in frontmatter, never in the directory structure, so
  links from the map to a closed ticket never rot.
- **Refer by name.** In everything the user reads — narration, the map's Decisions so far —
  call a ticket by its title, never by a bare number. `[Choose auth strategy](tickets/001-choose-auth-strategy.md)`
  reads at a glance; `#001` does not. The link rides inside the name.
- **One writer.** There is no claim or assignee field. Do not run two sessions against the
  same map concurrently.

## Map format

`map.md` is loaded once per session. It is an **index**, not a store: a decision lives in
exactly one place — its ticket — and the map only gists it and links.

```markdown
---
name: <map-slug>
status: charting | working | closed
---

# <Map title>

## Destination

<What reaching the end of this map looks like — the spec, decision, or change this effort
is finding its way to. One or two lines. Every session orients to it before choosing a ticket.>

## Notes

<Domain. Skills every session should consult. Standing preferences for this effort.
Any project-specific vocabulary established during charting.>

## Frontier

<!-- DERIVED. Regenerated on every close. Do not edit by hand. -->

- [<ticket title>](tickets/NNN-slug.md) — <type>

## Decisions so far

- [<closed ticket title>](tickets/NNN-slug.md) — <one-line gist of the answer>

## Not yet specified

<!-- Fog: in-scope questions not yet sharp enough to ticket. -->

## Out of scope

<!-- Ruled beyond the destination. Closed, never graduates. -->
```

The **Frontier** section is derived state, written down only so a human reading `map.md` on
a code host can see what is takeable without running an agent. Recompute it from the tickets
directory on every close; never trust it as a source of truth.

## Ticket format

`tickets/NNN-<slug>.md`:

```markdown
---
id: NNN
title: <Imperative phrase — "Choose auth strategy">
type: grilling | research | prototype | task
status: open | closed
blocked_by: [002, 005]
---

## Question

<The decision or investigation this ticket resolves. Sized to fit one agent session.>

## Answer

<!-- Empty until resolved. Written on close: the decision, and why. -->
```

A ticket is **unblocked** when every id in `blocked_by` has `status: closed`. The **frontier**
is every ticket that is open and unblocked. Compute it by scanning the directory:

```bash
grep -l "^status: open" docs/plans/<map-slug>/tickets/*.md
```

then filter by `blocked_by` against the closed set.

## Ticket types

Every ticket is either **HITL** — worked *with* the user, in live conversation — or **AFK**,
driven alone. Never answer the human's side of a HITL ticket yourself. A grilling that
invents its own answers has failed, silently, and poisoned every decision downstream.

| Type | Mode | How to resolve |
|---|---|---|
| `grilling` | HITL | Invoke `grill-me`. The default case. |
| `research` | AFK | Invoke the `research` skill. It reads and recommends; you decide and close. |
| `prototype` | HITL | Build a cheap, throwaway artifact to react to. See below. Link it; do not paste it. |
| `task` | either | Manual work that must happen before a decision can be made. |

**`task` is the one type that does rather than decides.** It earns its place only by
unblocking a decision — provisioning access so an API can be judged, moving data so its shape
can be seen. If a task would deliver part of the destination, it is not a task; it is a work
item, and it belongs after the map closes.

## Prototype tickets

A prototype is **throwaway code that answers a question**. Its purpose is to raise the
fidelity of the discussion — to give the user something concrete to react to, because the
useful signal is "wait, that shouldn't be possible," and that signal never arrives from prose.

**Build it in whatever the production artifact would be.** A PowerShell question gets a
PowerShell script. A Blazor question gets a page with switchable variants. A state-model
question gets a console app. Do not pick a prototype *format* and then translate the question
into it.

Before writing any of it, write the question at the top of the file. A prototype that answers
the wrong question is pure waste, and the question is what makes that checkable.

Put the answer-bearing logic behind a small pure **interface** — a reducer, a state machine,
a set of pure functions, whichever fits the question. That module is the part worth keeping;
the shell around it is disposable. Nothing flows from the shell into the module.

Rules:

- **Throwaway from day one**, and named so a casual reader can tell.
- **One command to run.** Whatever the project's runner is — `dotnet run`, a script path, a
  `just` recipe. The user must not have to remember a path.
- **In-memory state only**, unless persistence is itself the question.
- **No tests, no error handling beyond runnable, no abstractions, no "what if we later".**
  A prototype that needs tests has stopped being a prototype.
- **Surface the full state after every action** so the user sees exactly what changed.
- **UI variants must differ structurally** — layout, information hierarchy, primary affordance.
  Recolored variants are wallpaper. Three by default, five is the cap.

Hand it over and let the user drive it. Add actions they ask for; prototypes evolve.

On close, write the ticket's Answer — the judgment and the reasoning. Commit the prototype to
a throwaway branch and link it from the ticket as an asset. Main keeps the validated decision
and nothing else: prototype code left in the main branch rots fast and misleads the next reader.

## Fog of war

The map is deliberately incomplete. Do not chart what you cannot yet see.

**Not yet specified** holds the dim view: questions you can tell are coming but cannot phrase
sharply, because they hang on decisions still open. Resolving a ticket clears fog ahead of it,
graduating whatever became specifiable into fresh tickets.

The test is whether you can state the question precisely **now** — not whether you can answer it.

- **Ticket it** when the question is already sharp, even if it is blocked and unworkable today.
- **Leave it in the fog** when you cannot phrase it that sharply. Do not pre-slice fog into
  ticket-sized pieces; one patch may graduate into three tickets, or none.

## Out of scope

Fog gathers only *toward* the destination. Work beyond the destination is out of scope — not
fog — and gets its own section. It never graduates.

When an existing ticket turns out to sit past the destination, **close it** and add one line
to **Out of scope**: the gist, why it is out, and a link to the closed ticket. It does not
appear in **Decisions so far**, which records the route actually walked. A scope boundary is
not a step on the route.

---

# Modes

Determine the mode from what the user gave you. No map named, loose idea → **chart**. Map
named → **work**. Map named and no tickets open → **emit work items**.

**Never resolve more than one ticket per session, with the exception of research tickets.**
Resolving two decisions in one session means the second was made with a context window
already saturated by the first — which is the exact failure this skill exists to prevent.

## Mode A: Chart the map

### 1. Name the destination

Run `grill-me` to pin down what this map is finding its way to. The destination fixes the
scope, so it is settled before anything else. Write it as one or two lines describing an end
state, not an activity: "a spec for the sync engine that a fresh session can build from",
not "figure out syncing".

### 2. Map the frontier — breadth-first

This is a **different interview** from `grill-me`, which is deliberately depth-first. Do not
invoke it here. Run this instead:

> Interview the user one question at a time, fanning **across** the space rather than down
> into it. Each question must open an area not yet touched. When an answer exposes an
> interesting thread, **resist following it** — note it as fog and move laterally to the next
> untouched area. You are enumerating the open decisions, not resolving them. For each
> question, offer your recommended answer so the user can accept and keep moving.
>
> Stop when new questions stop surfacing new areas.

The failure mode is a deep spike into one thread with fog everywhere else. If you have asked
three consecutive questions about the same subsystem, you have drifted depth-first — pull out.

**If this surfaces no fog**, the way is already clear and the effort fits one session. Say so
and stop. Do not chart a map for work that does not need one.

### 3. Create the map

Write `docs/plans/<map-slug>/map.md` with Destination and Notes filled in, Decisions so far
and Frontier empty, and the fog sketched into **Not yet specified**. Set `status: charting`.

### 4. Create tickets, then wire

Two passes, because blocking references need ids to exist first:

1. Create every ticket you can specify now, with `blocked_by: []`.
2. Re-read them and fill in `blocked_by`.

Anything you cannot specify stays in the fog.

### 5. Fire research subagents

For every `research` ticket just created, invoke the `research` skill in parallel. Each
subagent writes one file to `docs/plans/<map-slug>/research/`, named after its ticket's slug.

A research subagent never closes its own ticket. When the findings return, weigh the
recommendation against the map's Destination and constraints — which the subagent never saw —
then write the ticket's Answer yourself and link the research file as an asset.

### 6. Stop

Charting is one session's work. Set `status: working` and hand-resolve nothing. Report the
frontier by name and stop.

## Mode B: Work the map

1. **Load `map.md`.** The low-resolution view only. Do not read every ticket body.
2. **Choose the ticket.** If the user named one, use it. Otherwise take the lowest-numbered
   ticket on the frontier.
3. **Resolve it.** Zoom as needed: fetch the full body of any related or closed ticket on
   demand. Use the type's method from the table above. When in doubt, `grill-me`.
4. **Record it.** Write the Answer section — the decision *and its reasoning*, because the
   reasoning is what a later session needs when it wants to know whether the decision still
   holds. Set `status: closed`. Append a one-line gist plus link to **Decisions so far**.
5. **Recompute the Frontier section.**
6. **Propagate.** Create any newly surfaced tickets (create, then wire). Graduate fog the
   answer made specifiable, clearing each graduated patch from **Not yet specified**. If the
   answer reveals a ticket sits past the destination, rule it out of scope rather than
   resolving it. If the decision invalidates other tickets, update or delete them.

## Mode C: Emit work items

Triggered when no open tickets remain. **Do this in a session that has the whole map loaded** —
decomposition is itself a design decision about batching, ordering, and what can run in
parallel, and it needs the full picture.

Decisions do not map one-to-one onto work items. One decision can spread across three work
items; three decisions can collapse into one; a decision that ruled something out of scope
produces none at all. Decompose by *unit of implementation work sized to one agent session*,
not by ticket.

`work/NNN-<slug>.md`:

```markdown
---
id: NNN
title: <Imperative phrase>
status: todo | done
implements: [001, 004]
depends_on: [002]
---

## Goal

<What this work item delivers.>

## Constraints

<The decisions this must honor, each linking the ticket that holds the full reasoning.>

## Acceptance criteria

<Observable, checkable. How an implementing agent knows it is finished.>
```

Keep `## Constraints` to the decision plus a link. An implementing agent that needs the *why*
follows the link. Do not restate the reasoning here — that duplicates it into a second place
where it will drift.

Set the map's `status: closed`. The user runs implementation sessions by pointing a fresh
agent at a single work file. Items with disjoint `depends_on` chains can run in parallel.