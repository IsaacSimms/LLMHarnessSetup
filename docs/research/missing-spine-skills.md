---
question: "Which skills, absent from this repo's on-disk inventory, would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine, and can be written as agent-neutral skill text shared by Cursor, Claude Code, and Grok?"
researched: 2026-10-30
confidence: partial
versions:
  - LLMHarnessSetup: d865677ae8c4f92b888335b0ed7c38033f951fe9
  - mattpocock/skills: a496f5513d4a4cf889bc48e189e579f4b0bdf6aa
  - Claude Code docs (code.claude.com/docs/en/skills, code.claude.com/docs/en/commands): fetched 2026-10-30; commands page describes bundled skills through Claude Code v2.1.309
---

# Which skills, absent from this repo's on-disk inventory, would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine, and can be written as agent-neutral skill text shared by Cursor, Claude Code, and Grok?

## Answer

Five absent skills clear the bar, all writable as shared text. The two large ones are a glossary-and-ADR discipline during design, and a diagnosis loop that builds a tight failing signal before any hypothesis. Three smaller ones cover a questionnaire when the person who knows is not in the session, an environment retro at recap time, and a human-only wizard for wayfinder task tickets.

## Findings

- The on-disk inventory is the root skill files plus `improve-codebase-architecture/`. `to-prd`, `diagnose`, `domain-modeling`, `diagnosing-bugs`, `to-questionnaire`, `retro`, and `wizard` are not among those files. `README.md` names `to-prd` and `diagnose` in the catalog only. — [T1] LLMHarnessSetup inventory (`README.md`, root `*.md`, `improve-codebase-architecture/`)
- `grill-me` ends in a chat-only settled-decisions log and does not write a file unless the user asks. The closing offer is `plan-as-artifact`, `wayfinder`, or stop. — [T1] `grill-me.md`
- `improve-codebase-architecture` says it is informed by the domain language in `CONTEXT.md` and by `docs/adr/`. While grilling a candidate it tells the agent to update `CONTEXT.md` and to offer an ADR using `grill-with-docs/CONTEXT-FORMAT.md` and `grill-with-docs/ADR-FORMAT.md`. Those format files are not in this repo. — [T1] `improve-codebase-architecture/SKILL.md`
- `build-context-doc` writes an architectural `Context.md` (stack, structure, entry points, commands). `recap-thread` can later merge durable facts into a root `CONTEXT.md` and into agent instruction files. `plan-as-artifact` stores per-plan Key Decisions. None of those three is an implementation-free glossary plus a spare-ADR rule applied while the terms are being decided. — [T1] `build-context-doc.md`, `recap-thread.md`, `plan-as-artifact.md`
- Upstream `domain-modeling` is the active discipline: challenge a term against `GLOSSARY.md`, sharpen fuzzy language, write the term down when it resolves, and offer an ADR only when the choice is hard to reverse, surprising without context, and a real trade-off. The glossary stays free of implementation detail. — [T1] mattpocock/skills `domain-modeling` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/domain-modeling/SKILL.md)
- Upstream `grill-with-docs` is the user-invoked composer. Its entire body is an instruction to invoke `grilling` and `domain-modeling`. — [T1] mattpocock/skills `grill-with-docs` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/grill-with-docs/SKILL.md)
- The upstream README states the repeated cost as agents inventing project jargon session after session, and names `grill-with-docs` as the skill that builds the shared language and ADRs. — [T2] mattpocock/skills README (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/README.md)
- `tdd` bug fixes enter at a failing test through the public interface, then a minimal fix. The skill does not require a tight red signal before hypothesizing, and it does not build a loop that is not that test. — [T1] `tdd.md`
- Upstream `diagnosing-bugs` forbids Phase 2 until one already-run command can go red on the user's exact symptom. The loop may be a test, an HTTP script, a CLI diff, a trace replay, a bisect, or a tightened flake rate. Later phases minimise, rank falsifiable hypotheses, instrument one variable at a time, and add a regression test only at a seam that can host the real bug. — [T1] mattpocock/skills `diagnosing-bugs` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/diagnosing-bugs/SKILL.md)
- At revision `a496f551` the tree has `diagnosing-bugs` and `to-spec`. It has no `skills/engineering/diagnose/SKILL.md` and no `skills/engineering/to-prd/SKILL.md`. The README catalog uses the current names. — [T1] mattpocock/skills git tree `a496f5513d4a4cf889bc48e189e579f4b0bdf6aa`; [T2] the same README
- Wayfinder `grilling` tickets are HITL. The skill says a grilling that invents the human's answers has failed and poisoned every decision downstream. `to-questionnaire` interviews the user only about who the document is for and what they need back, then writes a markdown questionnaire for that person. — [T1] `wayfinder.md`; [T1] mattpocock/skills `to-questionnaire` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/productivity/to-questionnaire/SKILL.md)
- `recap-thread` records the work and runs a context-file sweep for facts a future session would be wrong without. Upstream `retro` is a separate pass over navigation, automated checks, coding standards, steering-file no-ops, and tool cost, ordered by severity. — [T1] `recap-thread.md`; [T1] mattpocock/skills `retro` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/retro/SKILL.md)
- Wayfinder `task` tickets are manual work that unblocks a decision. Upstream `wizard` turns a human-only procedure into a confirmed, stage-by-stage bash script so the click-path is not re-explained, and it tells the agent not to invent UI steps it has not checked. — [T1] `wayfinder.md`; [T1] mattpocock/skills `wizard` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/wizard/SKILL.md)
- Wayfinder Mode C already emits session-sized work items with `depends_on` and acceptance criteria after a map closes. `plan-as-artifact` already orders phases with acceptance criteria. `tdd` already requires vertical tracer bullets once implementation starts. — [T1] `wayfinder.md`, `plan-as-artifact.md`, `tdd.md`
- `check-work` already runs builds and tests, checks plan acceptance criteria, and FAILs clear depth problems. Ambitious structural suggestions are explicitly suggestion-only, and the verifier text points them at a `code-review` skill that is not on disk. — [T1] `check-work.md`
- Upstream `codebase-design` restates the same module, interface, depth, seam, adapter, leverage, and locality vocabulary, including the deletion test and the one-adapter rule. This repo already has that vocabulary in `AGENTS.md` and `improve-codebase-architecture/LANGUAGE.md`, and the parallel interface pass in `INTERFACE-DESIGN.md`. — [T1] mattpocock/skills `codebase-design` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/codebase-design/SKILL.md); [T1] `AGENTS.md`, `improve-codebase-architecture/LANGUAGE.md`, `improve-codebase-architecture/INTERFACE-DESIGN.md`
- Claude Code documents bundled `/code-review`, `/simplify`, `/debug`, `/verify`, and `/run` as product skills, including effort levels, cloud review, and session debug logs. — [T2] Claude Code commands (https://code.claude.com/docs/en/commands); Claude Code skills (https://code.claude.com/docs/en/skills)

## Recommendation

- **1. domain-modeling.**
- **Cost removed.** Design sessions re-negotiate the same words, and hard-to-reverse trade-offs never become ADRs. `grill-me` keeps the settled log in chat. `improve-codebase-architecture` already tries to update a glossary and offer an ADR, then points at format files this repo does not have. A later recap or plan can store decisions, and still does not keep an implementation-free glossary or the three-part ADR test. Maps to `grill-me` and `improve-codebase-architecture`.
- **Change.** Add one shared skill with the behavior published as `domain-modeling`: challenge terms as they are used, record a resolved term immediately, and offer an ADR only when the choice is hard to reverse, surprising without context, and a real trade-off. `grill-with-docs` at this revision only composes that skill with grilling, so it is not a second addition. The shared text has to treat the glossary and ADR writes as docs under the same approval gate as other docs skills, so a design conversation still does not become implementation. Retargeting this repo's `CONTEXT.md` sentences onto that glossary is a prose edit, owned by `harness-prose-edits.md`.
- **Agent.** shared
- **Source.** [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/domain-modeling/SKILL.md ; gap in [T1] `grill-me.md` and `improve-codebase-architecture/SKILL.md` at `d865677ae8c4f92b888335b0ed7c38033f951fe9`

- **2. diagnosing-bugs.**
- **Cost removed.** On a hard bug the agent hypothesizes, or writes a unit test that is not the user's symptom, then patches the wrong thing. `tdd` starts the bug at a failing public-interface test and does not cover a loop that is not that test. Maps to `tdd`.
- **Change.** Add the shared loop published as `diagnosing-bugs`. No hypothesis until one command has already gone red on the user's exact symptom; then minimise, rank falsifiable hypotheses, instrument one variable at a time, and only then add a regression test at a seam that can host the bug. The README name `diagnose` is not a file in either repo at the revisions checked. The current text uses the name `diagnosing-bugs`.
- **Agent.** shared
- **Source.** [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/diagnosing-bugs/SKILL.md ; gap in [T1] `tdd.md`

- **3. to-questionnaire.**
- **Cost removed.** A wayfinder grilling ticket that invents the human's answers poisons every decision after it. `grill-me` only interviews the person in the session. When the fact sits with someone else, the repeated step is stalling or hand-drafting questions. Maps to `grill-me` and wayfinder `grilling` tickets.
- **Change.** Add a shared, user-invoked skill that asks who the document is for and what must be known afterward, then writes one markdown questionnaire for that person. A live grill stays the path when the user can answer.
- **Agent.** shared
- **Source.** [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/productivity/to-questionnaire/SKILL.md ; failure named in [T1] `wayfinder.md`

- **4. retro.**
- **Cost removed.** The same slow navigation, missing automated check, or dead steering rule returns next session. `recap-thread` records the work and may merge durable facts into context docs. It does not look for that environment miss. Maps to `recap-thread`.
- **Change.** Add a shared, user-invoked retro that reads the session and proposes environment changes in severity order: navigation, automated checks, judgement-call standards, steering no-ops, and tool cost. Items that are MCP, hooks, or edits to existing harness prose stay in `attached-capabilities.md` and `harness-prose-edits.md`. This addition is the procedure that surfaces them.
- **Agent.** shared
- **Source.** [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/retro/SKILL.md ; gap in [T1] `recap-thread.md`

- **5. wizard.**
- **Cost removed.** Human-only stages (a dashboard, a secret, a one-off cutover) get re-explained, or the agent invents a click-path. Wayfinder `task` tickets are the spine's place for manual work that unblocks a decision, and they stop at naming the task. Maps to wayfinder `task` tickets.
- **Change.** Add a shared skill that lists the human-only stages, confirms them, and authors a bash wizard for that run. Steps the agent can perform stay with the agent. The upstream skill also depends on `template.sh` in the same directory; that script was not opened, so the recommendation is the procedure in `SKILL.md`.
- **Agent.** shared
- **Source.** [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/wizard/SKILL.md ; gap in [T1] `wayfinder.md`

## Ruled out

- `grill-with-docs` as its own hit. At `a496f551` the file only invokes `grilling` and `domain-modeling`. The portable behavior is rank 1.
- `to-prd` and `diagnose` as names to install from `README.md`. The name in this repo's README is not a skill body. At `a496f551` the matching current files are `to-spec` and `diagnosing-bugs`. `diagnosing-bugs` is rank 2. `to-spec` is below.
- `to-spec`. `plan-as-artifact` already records the problem, requirements, decisions, and an optional testing strategy, and it stops without implementing. `to-spec`'s defining act is publishing that synthesis to an issue tracker. Destination: `left-field-workflows.md`.
- `to-tickets`. Wayfinder Mode C already emits session-sized work items with `depends_on`. `plan-as-artifact` already orders phases. `tdd` already requires vertical slices during implementation. The tracker's ticket graph is a different implementation sequence. Destination: `left-field-workflows.md`. The expand-contract exception inside that file, taken alone, would be a paragraph added to wayfinder or `tdd`. Destination for that fragment: `harness-prose-edits.md`.
- `implement` and `implement-spec`. They orchestrate `tdd`, then `code-review`, then a commit, and `implement-spec` adds concurrent worktrees on an integration branch. The spine already implements with `tdd` and offers `check-work` instead of auto-running it. Destination: `left-field-workflows.md`.
- `prototype`. Wayfinder already specifies throwaway prototypes in the production artifact, with the question written at the top, one command to run, and no tests. The upstream skill's logic branch is a single HTML file and its UI branch is route variants. That is a different prototype sequence. Destination: `left-field-workflows.md`.
- `code-review` (upstream) and this repo's dangling `code-review` pointer. `check-work` already checks the plan, the suite, and clear depth failures. The upstream skill is a standards-versus-spec diff report, including a Fowler smell baseline treated as judgement. Pointing the "ambitious code-judo" sentence at a real skill, or folding a two-axis report into `check-work`, is a behavior change to existing prose. Destination: `harness-prose-edits.md`.
- `codebase-design`, including its design-it-twice companion. The vocabulary and the parallel interface pass are already in `AGENTS.md`, `LANGUAGE.md`, and `INTERFACE-DESIGN.md`.
- `teach`. `learn` states that nothing carries between sessions and that it writes no files. A stateful multi-session teaching workspace is a different teaching sequence. Destination: `left-field-workflows.md`.
- `triage` and `ask-matt`. Issue-state routing, and a router over one author's user-invoked skills. `triage` is a different sequence. Destination: `left-field-workflows.md`. `ask-matt` has no spine step.
- `pr`. A pull-request body shape. No spine step in the list for this file.
- `setup-matt-pocock-skills`. One-time issue-tracker, label, and docs-path configuration. Destination: `attached-capabilities.md`.
- `writing-for-agents`. Not fetched. The README one-liner is a style guide for agent-facing documents. No spine failure was established from that line.
- Claude Code bundled `/code-review`, `/simplify`, `/debug`, `/verify`, and `/run`. Product commands (effort levels, cloud review, session debug logs, app launch). The bundled skill bodies were not opened. Destination: `attached-capabilities.md`. The narrower idea that `check-work` should observe the running app when a green suite is not enough is a behavior change to `check-work`. Destination for that idea: `harness-prose-edits.md`.
- Cursor Agent Skills docs and Codex skill-authoring docs, as returned by search. They describe portable skill packages. They named no missing spine workflow. — [T2] https://cursor.com/docs/skills ; https://learn.chatgpt.com/docs/build-skills

## Unverified leads

- Web search still shows `skills/engineering/diagnose/SKILL.md` and `skills/engineering/to-prd/SKILL.md` on `mattpocock/skills`, including a HEAD URL and older blobs `0288510dd61ff6ef7c2003834082ab8f2387e80e` and `16a2a5cd`. The git tree at `a496f551` does not contain those paths. The historical blobs and any rename commit were not opened, so those search hits are not evidence of what the old files said.

## Open questions

- Grok's first-party skill catalog was not found. Codex and Cursor results that were opened describe how skills are packaged, not a workflow list whose bodies were read. A skill that lives only in one of those catalogs could still be missing from this ranking.
- Peer skill repos other than `mattpocock/skills` were not opened.
- `wizard`'s `template.sh`, and the `GLOSSARY-FORMAT.md` / `ADR-FORMAT.md` siblings of `domain-modeling`, were not opened. The skill files describe when to write them. The exact templates are unread.
- Glossary and questionnaire writes happen during decision work. How that sits beside "plan mode edits nothing" is a decision for the session that adopts a skill, not a fact these sources settle. Aligning `improve-codebase-architecture`'s `CONTEXT.md` path with `GLOSSARY.md` is the prose file's job.
