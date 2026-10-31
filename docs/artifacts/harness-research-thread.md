---
title: Harness research thread
status: done
updated: 2026-10-30
project: LLMHarnessSetup
---

# Harness research thread

## TL;DR

Run one research thread that asks what would improve this harness, and write four review notes under `docs/research/`. The thread studies the live harness files and outward primary sources. It does not read `deprecated/`, and it does not edit skills, global instructions, or any other harness file. The end state is four markdown files, each with at most five ranked, cited recommendations, ready for a later human review. Adopting any recommendation is a separate decision.

## Problem Statement

This repo is the source of truth for a personal CLI-agent harness. Standing rules live in `AGENTS.md`. On-demand playbooks live in root skill files and in `improve-codebase-architecture/`. The same text is meant to be copied to Cursor, Claude Code, and Grok. The harness they run is Cursor. `README.md` names the workflow. `deprecated/` is a retired Pi tree and is not part of the harness.

The workflow on disk is:

1. `create-docs-folder` scaffolds `docs/` (`artifacts`, `recaps`, `handoffs`, `checks`, `plans`, `research`, `misc`) when asked.
2. `grill-me` stress-tests decisions.
3. `plan-as-artifact` writes an execution contract under `docs/artifacts/`.
4. Non-trivial implementation uses `tdd`.
5. `check-work` is offered after a non-trivial implement slice. It is not auto-run.
6. `wayfinder` is for work too large for one session, under `docs/plans/`.
7. `thread-handoff` continues work in a new session. `recap-thread` records work that is done.
8. `research` answers an external factual question and writes a cited note.
9. `build-context-doc` writes a project `Context.md`. `learn` teaches a tightly scoped skill. `improve-codebase-architecture` looks for deepening opportunities.

A design conversation is not permission to implement. Plan mode edits nothing until an explicit implement instruction.

What is missing is a cited answer to three kinds of gain, plus a fourth class that is not a skill file:

- Skills this repo does not have that would pay back on that spine.
- Edits to harness prose that would pay back on that spine.
- Ways of working that differ from that spine and would still be a large help.
- Capabilities you attach or configure besides skill text and standing instructions (MCP servers and peers).

`README.md` also names `to-prd` and `diagnose`. Those files are not in the repo. Treat them as absent.

### Inventory the subagents must treat as already present

Canonical harness prose:

| Path | Role |
|------|------|
| `AGENTS.md` | Global instructions on disk. Install copies are named `CLAUDE.md` and `GROK.md`. Those two filenames are not in this repo. |
| `README.md` | Human map of the harness, install targets, and spine. |
| `build-context-doc.md` | Skill. |
| `check-work.md` | Skill. |
| `create-docs-folder.md` | Skill. |
| `grill-me.md` | Skill. |
| `learn.md` | Skill. |
| `plan-as-artifact.md` | Skill. |
| `recap-thread.md` | Skill. |
| `research.md` | Skill. |
| `tdd.md` | Skill. |
| `thread-handoff.md` | Skill. |
| `wayfinder.md` | Skill. |
| `improve-codebase-architecture/SKILL.md` | Skill entry. |
| `improve-codebase-architecture/DEEPENING.md` | Skill companion. |
| `improve-codebase-architecture/INTERFACE-DESIGN.md` | Skill companion. |
| `improve-codebase-architecture/LANGUAGE.md` | Skill companion. Ubiquitous language: Module, Interface, Implementation, Depth, Seam, Adapter, Leverage, Locality, Scan. |

Ignored. Do not open, cite, compare, or recommend edits to anything under this path:

- `deprecated/` — retired Pi harness, including `deprecated/PiSetup/` and `deprecated/APPEND_SYSTEM(globalInstrucDocforPI).md`

Also not edit targets, and not worth search effort:

- `.gitattributes`
- `.git/`
- `.cursor/` (empty)
- `docs/` (this plan and the future findings live here; they are not edit targets)

Install targets, from `README.md`, are sync destinations. They are not files in this repo:

- Claude Code: `~/.claude/CLAUDE.md`, `~/.claude/skills/`
- Grok: `~/.grok/rules/GROK.md`, `~/.grok/skills/`
- Cursor CLI: `~/.cursor/skills/<skill>/SKILL.md`

Policy already written in the repo: skill bodies stay agent-neutral, and the shared text stays identical across those installs. Platform-only skills may exist only in one agent's install and are not required in this repo.

## Requirements

1. The executing thread must spawn four research passes, one per brief in this plan. When the runtime can spawn background subagents, the four passes run in parallel. Each pass writes exactly one file:
   - `docs/research/missing-spine-skills.md`
   - `docs/research/harness-prose-edits.md`
   - `docs/research/left-field-workflows.md`
   - `docs/research/attached-capabilities.md`
2. Each file must use the findings format in Constraints. `researched` is the date the pass finishes. `question` is the brief's question, verbatim.
3. Each file's `## Recommendation` contains at most five items, ranked. Fewer is required when fewer clear that file's bar. Padding to five fails the file.
4. A hit in `missing-spine-skills.md`, `harness-prose-edits.md`, or `attached-capabilities.md` clears the bar only when it names all of: the repeated failure, repeated manual step, or mishandled decision; the spine step or existing inventory path it maps to; a primary-source citation with tier. A hit that cannot name the step or path does not clear the bar.
5. A hit in `left-field-workflows.md` clears the bar when it names the idea, why it might help, and a primary-source citation with tier. It does not have to show that the current spine cannot do the job.
6. Rank within a file by the size of the repeated cost removed. For the left-field file, rank by the size of the help claimed. Break ties by source tier: T1, then T2, then T3.
7. Every ranked hit cites a T1, T2, or T3 source. The citation is enough for a later reader to open that source. Forum-only claims appear only under `## Unverified leads`, and they are not ranked.
8. `missing-spine-skills.md` and `harness-prose-edits.md` recommend only changes that can be written as agent-neutral text shared by Cursor, Claude Code, and Grok. A one-agent mechanism that is not an attachable is ruled out of those two files and named as belonging in `left-field-workflows.md`.
9. `attached-capabilities.md` may name the agent. Its hits are things you attach or configure besides skill text and standing instructions. The class is MCP servers, hooks, rules, plugins, custom tools, slash commands, subagent presets, permission or sandbox presets, and peers of that class.
10. `harness-prose-edits.md` recommends behavior changes to the canonical prose in the inventory. Nothing under `deprecated/` is a target.
11. A missing-skill hit must be absent from the on-disk inventory. A prose-edit hit must change behavior. A rephrase that leaves behavior the same is ruled out.
12. Recommendations name the change and the cost it removes. They do not include a replacement skill body, a patch, or a rewritten `AGENTS.md`.
13. After the four files exist and meet the phase acceptance criteria, the parent reads those files, does not re-read their sources, and reports in chat: each path, each confidence, the ranked names, and any candidate one file ruled out as belonging in another file. The parent does not adopt a recommendation and does not add a fifth markdown file.
14. If the runtime cannot spawn background subagents, the parent runs the four briefs itself, in order, and says in each file and in the chat report that context isolation was lost.

## Constraints

- Do not edit, create, rename, or delete harness prose. The only new files are the four findings files. The executing agent may update Progress and the frontmatter `status` and `updated` on this plan file.
- Do not open, list, cite, or recommend changes to `deprecated/`. A path under that folder is not T1, not an edit target, and not evidence. If another file points at it, do not follow the pointer.
- Do not sync installs (`~/.cursor`, `~/.claude`, `~/.grok`).
- Do not implement a recommended skill, edit, workflow, or attachable.
- Do not re-open Key Decisions.
- Plan mode and explicit implement approval are not fences on what a finding may *say*. They are fences on what this thread may *do*. The thread must not change those rules in the repo.
- Source tiers, and only these count as evidence:
  - **T1** — the thing itself. Skill files in the inventory, a GitHub skill repo once the pass has read the skill file, a product's schema or reference implementation. Files under `deprecated/` are not T1.
  - **T2** — first-party docs, changelogs, release notes, migration guides.
  - **T3** — a maintainer's own statement in an issue, PR, RFC, or design doc.
- Reddit, forums, blogs, Stack Overflow, tutorials, courses, and aggregator posts may be used to find a primary source. They are not citations for a ranked hit. When a claim rests only on one of those, it goes under `## Unverified leads`.
- Search is wide. It includes model-specific and product-specific material, forums as finders, and GitHub skill repos in the Matt Pocock tradition and peers discovered from there. It includes official material for Cursor, Claude Code, Codex, and Grok when first-party skill or instruction docs exist. It does not research the Pi harness, and it does not require a survey of every coding-agent product.
- This repo's listed files are T1 for what the harness already is. Outward sources are T1–T3 for what exists elsewhere. Key Decisions in this plan are constraints, not evidence.
- Read-only toward every system except the one findings file a pass is assigned. No writes, mutations, or deletes elsewhere. No production endpoints. No credential creation. If a question cannot be answered inside those limits, the file says so.
- Stop a pass when its question is answered, or when three consecutive sources add nothing new. Hard ceiling about 20 source fetches for that pass. If the ceiling hits with the question unanswered, set `confidence: inconclusive` and do not pad.
- One find has one owning file. An attachable belongs in `attached-capabilities.md`. A different sequence of work belongs in `left-field-workflows.md`. A missing shared skill belongs in `missing-spine-skills.md`. A behavior change to existing prose belongs in `harness-prose-edits.md`. A pass that finds a candidate for another file lists it under `## Ruled out` with the destination filename. It does not also rank that candidate.
- Attachable exclusions: choosing or shopping for models, billing, editor themes, and git hosting.
- Do not treat `to-prd` or `diagnose` as installed. A recommendation to add either still has to clear the missing-skill bar and cite a primary source for what the skill is. The name in `README.md` is not that source.

### Findings format

Every findings file:

```markdown
---
question: <the brief question, verbatim>
researched: <YYYY-MM-DD>
confidence: confident | partial | inconclusive
versions:
  - <library/API/spec/repo>: <version or revision checked>
---

# <Question as a heading>

## Answer

<Two or three sentences. Direct answer. If nothing clears the bar, say so.>

## Findings

- <Claim.> — [T1] <source name>(<url or path>)
- <Claim.> — [T2] <source name>(<url or path>)

## Recommendation

<At most five ranked hits. Use the hit shape for that file. Omit the section's list when nothing clears the bar, and say that.>

## Ruled out

<Candidates eliminated, with the reason. Wrong-file items name the destination file. Sources that added nothing are listed here too.>

## Unverified leads

<Forum-only or other non-primary leads. Not ranked. Name the lead, where it was seen, and which primary source is missing. Write "None." when there are none.>

## Open questions

<What the pass could not resolve.>
```

Hit shape for `missing-spine-skills.md`, `harness-prose-edits.md`, and `attached-capabilities.md`:

- **Rank and name.**
- **Cost removed.** The repeated failure, manual step, or mishandled decision, and the spine step or inventory path it maps to.
- **Change.** What would be added or edited, in a few sentences. No skill body and no patch.
- **Agent.** `shared` for the first two files. For attachables, the product name when the capability is product-specific, otherwise `shared`.
- **Source.** Tier and a locator a later reader can open.

Hit shape for `left-field-workflows.md`:

- **Rank and name.**
- **Why it might help.**
- **Agent.** `shared`, or the product name when the idea only exists on one agent.
- **Source.** Tier and a locator a later reader can open.

### Shared context for every brief

Paste this into each subagent's context along with that brief's extra lines:

> This repo (`LLMHarnessSetup`) is the canonical harness for Cursor CLI, Claude Code, and Grok. The harness they run is Cursor. Do not open `deprecated/`. It is a retired Pi tree and is not a source, not an edit target, and not evidence. The spine is: docs scaffold, grill, plan artifact, implement with TDD, offered check. Wayfinder when the work does not fit one session. Handoff or recap at the session edge. Research for external facts. Build-context-doc, learn, and improve-codebase-architecture sit beside that spine. A design conversation is not implement permission. Plan mode edits nothing. Skill text is supposed to stay agent-neutral and identical across Cursor, Claude, and Grok. The inventory is in `docs/artifacts/harness-research-thread.md`. Read those files as T1 for what already exists. Cite outward T1–T3 sources for what exists elsewhere. Do not edit the repo. Write only the assigned findings file. Do not design a skill body or a patch. Do not treat this plan's decisions as evidence.

## Key Decisions

> **Decision:** The artifact is this prompt. The research thread writes review notes and does not design or apply skill edits.
> **Rationale:** The wanted output is a review set with a fixed bar, produced before any harness change.
> **Alternatives considered:** An outcome contract with no research questions. A research brief that left "better" undefined. Designing the skill patches in this plan.

> **Decision:** The surface under study is this repo's shared harness. Cursor, Claude, and Grok are sync targets of that text.
> **Rationale:** The repo already says to edit here first and keep the copies identical.
> **Alternatives considered:** Research only the live Cursor install and backport. Treat Pi as a divergent harness. Allow each agent its own variant.

> **Decision:** Primary gain is spine payback. A find clears that bar when it removes a repeated failure, a repeated manual step, or a mishandled decision on the spine above. Left-field gain is separate and lighter: name the idea and why it might help.
> **Rationale:** The biggest keys are outward finds that pay back on the workflow already in use. A different way of working stays in scope when the help is large, without forcing it through the spine bar.
> **Alternatives considered:** Ecosystem-first, where a popular skill qualifies with no named cost. Pain-first, with no outward scan. Requiring left-field hits to prove the spine cannot express the job.

> **Decision:** Plan mode and explicit implement approval are parked as design fences. Findings may discuss them. This thread must not change them in the repo.
> **Rationale:** The grill's job became the research prompt. Whether those two rules should ever loosen is a review decision after the notes exist.
> **Alternatives considered:** Forbid findings that touch either rule. Allow the research thread to edit the rules.

> **Decision:** Four files, four questions, at most five ranked hits each.
> **Rationale:** The research skill wants one narrow question per pass. Missing skills, prose edits, left-field workflows, and attachables are different questions.
> **Alternatives considered:** One broad file. Two files that blend new skills with edits. Folding MCP and peers into the left-field file. A fourth file limited to MCP servers. A fifth file for platform-only skills.

> **Decision:** Files 1 and 2 stay agent-neutral. A one-agent mechanism that is not an attachable belongs in the left-field file. The attachables file may name the agent.
> **Rationale:** Shared skill text is one body. Attachables are often product-specific, and that file is where those recommendations belong.
> **Alternatives considered:** Let files 1 and 2 recommend Cursor-only or Claude-only mechanisms. Add another file only for platform-only finds.

> **Decision:** Prose-edit targets are the canonical harness prose outside `docs/`. Pi copies of root skills count once, against the root file. Binaries, `models-store.json`, `settings.json`, and git metadata are out.
> **Rationale:** That is the text the agents follow. Recommending the same edit against a Pi copy double-counts it. Binaries and JSON are not the harness.
> **Alternatives considered:** Root skills, `improve-codebase-architecture`, and `AGENTS.md` only. Any path outside `docs/`, including binaries and JSON.

> **Decision:** Search covers model-specific material, forums, and GitHub skill repos (Matt Pocock and peers). Ranked hits cite a skill repo, official docs, or a maintainer's own statement. A forum-only claim is an unverified lead outside the top five.
> **Rationale:** Forums are how a find shows up. The note has to point at something a later reader can open.
> **Alternatives considered:** Cite Reddit as evidence. Do not search forums. Limit the search to Cursor and Claude Code official docs. Require a survey of every coding-agent product.

> **Decision:** Rank by the size of the repeated cost removed. Break ties by source tier, T1 then T2 then T3. The left-field file ranks by the size of the help claimed, with the same tie-break.
> **Rationale:** The top five should be the biggest keys, and a stronger source wins a tie.
> **Alternatives considered:** No rank key. Cap of three. No cap.

> **Decision:** Attachables are their own file and use the spine-payback bar. The class includes MCP servers, hooks, rules, plugins, custom tools, slash commands, subagent presets, permission or sandbox presets, and peers of that class. Model shopping, billing, editor themes, and git hosting are out.
> **Rationale:** Those are harness things you can give an agent that are not skill files and not global instructions. The left-field bar is too weak for them, and the prose-edit file cannot express them.
> **Alternatives considered:** Leave them in the left-field file. Research MCP servers only.

> **Decision:** The thread is done when the four findings files meet the requirements and the parent has reported them in chat. No fifth synthesis file. No harness diff.
> **Rationale:** The review set is the four notes. Adoption is later.
> **Alternatives considered:** One combined markdown file. Parent rewrites the four notes into a single ranking.

> **Decision:** Ignore `deprecated/` completely. This supersedes every earlier instruction to read, compare, cite, or edit `PiSetup/`, `APPEND_SYSTEM`, or a local Pi copy.
> **Rationale:** The Pi harness is retired and now lives under `deprecated/`. The harness they run is Cursor. Studying that folder would spend the review on a tree they do not use.
> **Alternatives considered:** Keep the Pi copies as a comparison inventory. Keep researching Pi's own docs as a required ecosystem.

## Deferred

Adopting any recommendation. Writing or editing skill bodies, `AGENTS.md`, or other harness prose. Syncing Cursor, Claude, and Grok installs. Choosing which hits to implement. A combined synthesis note.

## Implementation Phases

### Phase 1 — Write the four findings files

**Objective:** Four cited review notes exist, one per brief, and each meets the findings format and the bar for its question.

**Tasks:**

1. Read Constraints, Key Decisions, the inventory, and the four briefs. Do not re-open them.
2. Spawn four background subagents in parallel when the runtime supports it. Give each subagent only its brief: the question, the shared context, that brief's extra lines, the output path, and the findings format. Tell each subagent to write exactly one file, to edit nothing else, and to return the path plus a three-line summary.
3. If the runtime cannot spawn subagents, run the four briefs in the parent session in brief order, and record the context-isolation loss as required above.
4. Check each file against the phase acceptance criteria. Re-run only a failing brief, once. If it still fails, leave the file and record what failed in the Phase 2 report.

**Brief — missing skills**

- **Output:** `docs/research/missing-spine-skills.md`
- **Question:** Which skills, absent from this repo's on-disk inventory, would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine, and can be written as agent-neutral skill text shared by Cursor, Claude Code, and Grok?
- **Extra context:** `README.md` names `to-prd` and `diagnose`, and those files are not on disk. They are not installed skills. A one-agent mechanism that cannot be written as shared neutral text is ruled out of this file; name `left-field-workflows.md` or `attached-capabilities.md` as the destination. Do not recommend a skill the inventory already has.

**Brief — harness prose edits**

- **Output:** `docs/research/harness-prose-edits.md`
- **Question:** Which behavior-changing edits to this repo's canonical harness prose would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine, while remaining agent-neutral text shared by Cursor, Claude Code, and Grok?
- **Extra context:** Edit targets are the inventory's canonical prose only. Do not open or target `deprecated/`, `.gitattributes`, `.git/`, `.cursor/`, or anything under `docs/`. A wording change that does not change behavior is ruled out. Do not include a patch or a rewritten file.

**Brief — left-field workflows**

- **Output:** `docs/research/left-field-workflows.md`
- **Question:** Which ways of working, different from this repo's current spine, would be a large help? Name each idea and why it might help.
- **Extra context:** This file does not have to prove the spine cannot express the idea. Attachables (MCP, hooks, rules, plugins, tools, slash commands, subagent presets, permission presets, and peers) belong in `attached-capabilities.md` even when they feel new. Missing shared skills belong in `missing-spine-skills.md`. Edits to existing prose belong in `harness-prose-edits.md`. A one-agent way of working that is not an attachable belongs here.

**Brief — attached capabilities**

- **Output:** `docs/research/attached-capabilities.md`
- **Question:** Which attached or configured capabilities, other than skill text and standing global instructions, would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine?
- **Extra context:** In scope: MCP servers, hooks, rules, plugins, custom tools, slash commands, subagent presets, permission or sandbox presets, and peers of that class. Name the agent when the capability is product-specific. The harness they run is Cursor. Out of scope: model shopping, billing, editor themes, git hosting, new skill text, edits to harness prose, and anything under `deprecated/`. New skill text and prose edits belong in the missing-skill and prose-edit files.

**Acceptance criteria:**

- All four output paths exist.
- Each file has the required frontmatter and the sections `Answer`, `Findings`, `Recommendation`, `Ruled out`, `Unverified leads`, and `Open questions`.
- Each `Recommendation` has five or fewer hits, and every hit uses that file's hit shape and a T1, T2, or T3 citation.
- No hit is a forum-only claim. Forum-only items are under `Unverified leads` or absent.
- No file contains a skill body or a patch.
- No findings file cites or describes a path under `deprecated/`.
- No harness prose path has a diff from this research thread.
- A brief that could not be re-run successfully is named, with the failed criterion, in the Phase 2 report.

### Phase 2 — Report the review set

**Objective:** The human gets a chat report that points at the four notes and does not decide among them.

**Tasks:**

1. Read the four findings files. Do not re-read their sources.
2. Report each path, confidence, and ranked names. Name any candidate one file marked as belonging in another file, and say whether that destination file already contains it.
3. State whether any pass lost context isolation or failed acceptance after its single re-run.
4. Update this plan's Progress. Set frontmatter `status` to `done` and refresh `updated` only when every acceptance criterion above is met. If a criterion failed, leave `status` at `in-progress` and say which criterion failed.
5. Stop. Do not edit harness prose. Do not add a synthesis file. Do not ask which hit to implement.

**Acceptance criteria:**

- The chat report includes the four paths, four confidence values, and the ranked names from each file that has any.
- The report names cross-file misfiles the notes themselves recorded.
- The git diff for the thread contains no changes outside `docs/research/*.md` and this plan's Progress and frontmatter.

## Progress

- [x] Phase 1 — Write the four findings files
  - [x] Read Constraints, Key Decisions, inventory, and briefs
  - [x] Run the four briefs (parallel subagents, or inline with the isolation loss recorded)
  - [x] `docs/research/missing-spine-skills.md` meets the findings acceptance criteria
  - [x] `docs/research/harness-prose-edits.md` meets the findings acceptance criteria
  - [x] `docs/research/left-field-workflows.md` meets the findings acceptance criteria
  - [x] `docs/research/attached-capabilities.md` meets the findings acceptance criteria
  - [x] Re-run any single failing brief once
- [x] Phase 2 — Report the review set
  - [x] Read the four files without re-reading their sources
  - [x] Report paths, confidence, ranked names, and cross-file misfiles
  - [x] Update Progress and frontmatter to match what actually passed
  - [x] Leave harness prose unchanged

## Cold-start

Read `docs/artifacts/harness-research-thread.md` and continue from the plan. Honor Constraints and Key Decisions; do not re-open them. Where an earlier Key Decision mentions Pi copies, the later decision to ignore `deprecated/` wins. Start at the first incomplete Progress item. There are no Open Questions. This plan file alone is not permission to edit the harness. Running the research thread still needs the user to say to execute this plan.

## Reference Files

- `docs/artifacts/harness-research-thread.md` — this plan
- `README.md` — spine, install targets, skill catalog
- `AGENTS.md` — global instructions on disk
- `research.md` — the research skill as kept in this repo
- Root skill files and `improve-codebase-architecture/` — current skill inventory

## Revision Notes

- 2026-10-30 — Pi harness moved to `deprecated/`. The research thread ignores that folder. It is not inventory, not a citation source, and not an edit target. Earlier decisions that name `PiSetup/` or the Pi append-system file are superseded for this thread.
