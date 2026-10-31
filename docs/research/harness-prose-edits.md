---
question: "Which behavior-changing edits to this repo's canonical harness prose would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine, while remaining agent-neutral text shared by Cursor, Claude Code, and Grok?"
researched: 2026-10-30
confidence: confident
versions:
  - LLMHarnessSetup: d865677ae8c4f92b888335b0ed7c38033f951fe9
  - mattpocock/skills: a496f5513d4a4cf889bc48e189e579f4b0bdf6aa
  - Claude Code docs (memory, permission-modes, skills): code.claude.com pages fetched 2026-10-30
  - Cursor docs (skills, subagents): cursor.com/docs pages fetched 2026-10-30
---

# Which behavior-changing edits to this repo's canonical harness prose would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine, while remaining agent-neutral text shared by Cursor, Claude Code, and Grok?

## Status

Done 2026-10-30. All five recommendations below were applied. Hits 1–3 were also copied into `~/.cursor/skills/`. Hits 4–5 are only in this repo; that skill has no user-context copy. The findings below still describe the prose as it was when researched.

## Answer

Five edits clear the bar. The largest stops a recap from replacing standing instruction files. The others stop an ideation handoff from authorizing implementation, make a grill's settled log the plan in the same turn, and stop the architecture skill from following missing format files or scanning past a named area. Each one is shared prose. Adoption is a later decision: every edit also drops a side effect the current text performs on purpose.

## Findings

- `recap-thread.md` merges durable facts into `AGENTS.md`, `CLAUDE.md`, `GROK.md`, and the other instruction names it lists, in the same run, without asking. A new fact that contradicts an existing line replaces that line. The sweep stops to ask only for a listed safety rule, or when two context docs already disagree. — [T1] recap-thread.md (Context file sweep)
- Claude Code treats `CLAUDE.md` and `AGENTS.md` as instructions the user writes. Claude's own notes go to auto memory. Instruction files are targeted under 200 lines because longer files reduce adherence. Contradicting instructions are picked arbitrarily. `/doctor prompt-audit` proposes edits and changes nothing until the user asks to apply them. `/init` suggests improvements when a `CLAUDE.md` already exists, rather than overwriting it. — [T2] Claude Code memory (https://code.claude.com/docs/en/memory)
- `thread-handoff.md` Mode B sets the receiving job to "Pressure-test this design, then implement," and the paste-ready opener tells the next session to implement once the spec holds up. "Holds up" is the receiving agent's judgment. — [T1] thread-handoff.md (Mode B, Step 7, Mode B opener)
- `AGENTS.md` requires an explicit implement approval. A design discussion is not that approval. — [T1] AGENTS.md (Approval & Plan Mode)
- Claude Code plan mode blocks source edits until the user approves the plan from an explicit prompt: approve and start editing, or stay in planning. — [T2] Claude Code permission modes (https://code.claude.com/docs/en/permission-modes)
- The upstream handoff skill writes a continuation document. The next session's focus is an argument the user passes. The skill text does not tell that session to implement. — [T1] mattpocock/skills handoff (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/productivity/handoff/SKILL.md)
- `grill-me.md` emits the settled-decisions log in chat only, then offers `plan-as-artifact`, `wayfinder`, or stop. The offer does not say that a selection runs that skill, and it does not require the log to be the plan's Key Decisions. — [T1] grill-me.md (Closing the grill)
- `plan-as-artifact.md` already requires every resolved grill branch in Key Decisions, and it says writing the plan is not permission to implement. — [T1] plan-as-artifact.md (Key Decisions, Step 3)
- The maintainer docs for the docs-writing grill say decisions that are not written down stay in the conversation, and later steps soften precise answers into weaker prose. That page calls this the most serious open complaint, and it says the close is often open-ended. The same-conversation mitigation it names is to write the spec immediately, then re-read it against the answers. — [T2] mattpocock/skills grill-with-docs docs (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/docs/engineering/grill-with-docs.md)
- `improve-codebase-architecture/SKILL.md` tells the agent to add terms to `CONTEXT.md` using `../grill-with-docs/CONTEXT-FORMAT.md`, to create that file lazily, and to offer an ADR using `../grill-with-docs/ADR-FORMAT.md`. Those two format files are not in this repo. The explore step then says to use the Agent tool with `subagent_type=Explore` and to explore organically, with no step that keeps a named direction and skips the rest. — [T1] improve-codebase-architecture/SKILL.md (Explore, Grilling loop)
- `build-context-doc.md` writes `Context.md` at the repo root as an architecture reference (stack, seams, API, dev commands). On a case-sensitive filesystem that is a different path from `CONTEXT.md`. — [T1] build-context-doc.md (Phase 2)
- The maintainer ADR for this skill family classes `improve-codebase-architecture` as a soft dependency: refer to the project's domain glossary and nearby ADRs in vague prose. If those docs are absent, the skill still runs and the output is only less sharp. A hard pointer to missing setup belongs on skills that cannot function without it. — [T3] mattpocock/skills ADR 0001 (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md)
- The current upstream architecture skill's first process step is "Scope before you scan." A direction the user named is the scope. Otherwise the scan starts from recent commit history and widens only when that history has no hot spot. — [T1] mattpocock/skills improve-codebase-architecture (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/improve-codebase-architecture/SKILL.md)

## Recommendation

Ranked by the size of the repeated cost. The tradeoff on each hit is the same shape: the current text spends that cost to do something automatic (rewrite rules, start coding, keep the log in chat, invent a glossary, scan the whole tree), and the edit makes that automatic step wait for an explicit ask or for inputs that already exist.

- **1. Recap leaves standing instructions alone.**
- **Cost removed.** A finished thread rewrites the files that gate the spine. `recap-thread.md` replaces a contradictory line in `AGENTS.md`, `CLAUDE.md`, or `GROK.md` without asking, except for a short safety list. A session-scoped conclusion can become the next session's TDD, approval, or workflow rule, and the human diffs or reverts it. Maps to `recap-thread` and the `AGENTS.md` standing rules.
- **Change.** The context sweep in `recap-thread.md` still updates the project context file with architecture and project facts. Standing instruction files stay unchanged unless the user explicitly asked to change standing rules. A contradiction with those files is named in the closing blurb and left in place. The human asks when a session fact should become a standing rule.
- **Agent.** shared
- **Source.** [T2] Claude Code memory, "Who writes it" and the prompt-audit rule (https://code.claude.com/docs/en/memory)

- **2. Ideation handoff stops before implementation.**
- **Cost removed.** Every Mode B handoff tells the next session to implement once the spec "holds up," which is that session's own review. `AGENTS.md` says a design discussion is not implement permission, so the human stops the agent or reverts the edits. Maps to `thread-handoff` and the `AGENTS.md` approval rule.
- **Change.** In `thread-handoff.md`, Mode B's receiving job, next-step line, and paste-ready opener end at the pressure test. The new session waits for an explicit implement instruction before writing code. Mode A stays resume-and-continue for work that is already implementation.
- **Agent.** shared
- **Source.** [T2] Claude Code permission modes, plan mode blocks edits until the user approves the plan (https://code.claude.com/docs/en/permission-modes). The same boundary is what the upstream handoff skill actually does: [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/productivity/handoff/SKILL.md

- **3. Grill selection writes the plan from the settled log.**
- **Cost removed.** `grill-me.md` keeps the settled log in chat and does not say that picking `plan-as-artifact` or `wayfinder` runs that skill. The human asks a second time, or a later plan softens branches that were already decided. Maps to `grill-me` and `plan-as-artifact`.
- **Change.** In `grill-me.md`, the closing selection is the request. `plan-as-artifact` runs in that turn, and its Key Decisions are the settled log, every branch. `wayfinder` runs the same way when that option is the selection. Stop writes nothing. `plan-as-artifact.md` checks those Key Decisions against the log before save, which is the re-read the maintainer docs require of the written artifact. Writing the plan remains not permission to implement.
- **Agent.** shared
- **Source.** [T2] mattpocock/skills grill-with-docs docs, "Where did all my other decisions go?" and the open-ended close (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/docs/engineering/grill-with-docs.md)

- **4. Architecture review no longer follows missing format files.**
- **Cost removed.** On every deepening conversation that names a term or rejects a candidate, `improve-codebase-architecture/SKILL.md` sends the agent to `grill-with-docs` format files that are not in this repo, and it creates `CONTEXT.md` lazily. The agent invents a format, or the human supplies one. On Linux that new file sits beside the `Context.md` that `build-context-doc.md` already writes. Maps to `improve-codebase-architecture` and `build-context-doc`.
- **Change.** The grilling loop reads a glossary and ADRs when they already exist, and the review continues when they do not. It stops opening `../grill-with-docs/CONTEXT-FORMAT.md` and `ADR-FORMAT.md`, stops creating `CONTEXT.md`, and stops appending glossary terms to `Context.md`. An ADR stays a question in chat unless this package itself contains the format.
- **Agent.** shared
- **Source.** [T3] mattpocock/skills ADR 0001 (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md). Current skill text that already behaves this way: [T1] https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/improve-codebase-architecture/SKILL.md

- **5. Architecture scan takes a scope first.**
- **Cost removed.** The explore step says to walk the codebase organically. A direction the user already gave is not binding, so the human repeats the subsystem after a whole-repo pass. Maps to `improve-codebase-architecture`.
- **Change.** A named direction is the whole scan. With no named direction, the scan starts from recent commit history and widens only when that history has no hot spot. The unscoped walk stops being the first move.
- **Agent.** shared
- **Source.** [T1] mattpocock/skills improve-codebase-architecture, "Scope before you scan" (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/improve-codebase-architecture/SKILL.md)

## Ruled out

- Adding `to-spec`, `to-prd`, `diagnose`, `grill-with-docs`, `domain-modeling`, or `code-review`. Those names are absent from the on-disk inventory (`README.md` lists `to-prd` and `diagnose`; `check-work.md` routes ambitious review to `code-review`). Destination: `missing-spine-skills.md`.
- Claude auto memory as the place session learnings should go. That store is a Claude Code product feature. Destination: `attached-capabilities.md`. The shared-prose change is hit 1.
- Replacing `grill-me.md`'s structured picker with the upstream grilling shape (the whole frontier in one prose round, worded so "yes" accepts the recommendation). That is a different interview sequence. `grill-me.md` already falls back when the host has no `AskQuestion` tool. Destination: `left-field-workflows.md`.
- HTML architecture reports, temp-dir handoffs, and moving refactoring out of the red-green loop into `code-review`. The first is a presentation change with no repeated failure in the local numbered list. The second abandons `docs/handoffs/`. The third removes the refactor step `tdd.md` and `AGENTS.md` require, and it depends on a skill this repo does not have.
- Dropping the TDD seam-confirmation ask. The upstream `tdd` skill still requires the user to confirm seams before any test is written (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/tdd/SKILL.md).
- Waiting for the user before design-it-twice sub-agents. Local `improve-codebase-architecture/INTERFACE-DESIGN.md` and upstream `DESIGN-IT-TWICE.md` both show the frame and then spawn immediately (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/codebase-design/DESIGN-IT-TWICE.md).
- Stopping the docs-path question when `docs/` is missing from cwd. The ask is how `plan-as-artifact.md`, `thread-handoff.md`, `recap-thread.md`, `check-work.md`, and `create-docs-folder.md` handle two real candidates. No outward source showed that confirming a single default is the failure.
- Rewording `improve-codebase-architecture/SKILL.md` and `INTERFACE-DESIGN.md` off the literal `Agent` tool and `subagent_type=Explore`. Upstream now says "spawn a sub-agent with a fresh context window," and Cursor's built-in explore subagent is documented (https://cursor.com/docs/subagents). A repeated tool-call failure on the harness they run was not established. `check-work.md` already has an in-process fallback the architecture files lack; that gap stays an open question for hosts with no subagent tool.
- Typo and comment-style edits (`canidate`, "A ood test", the `(UL)` marker, duplicating the path-resolution steps). They leave behavior the same.
- Wayfinder's instruction to commit a prototype to a throwaway branch. No standing rule in the canonical prose forbids a commit, so this pass had no visible contradiction to remove.

Sources that did not add a further hit: Anthropic skill-authoring best practices (https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) on one-level-deep references; Claude Code skills docs (https://code.claude.com/docs/en/skills) on which frontmatter is the open standard; Cursor skills docs (https://cursor.com/docs/skills) on portable `SKILL.md` text. They support agent-neutral wording and were not a sixth cost.

## Unverified leads

None.

## Open questions

- Grok's first-party skill and tool docs were not fetched. Whether Grok errors on `AskQuestion`, `argument-hint`, or `subagent_type=Explore` is unknown. The five hits do not depend on that.
- This pass did not call Cursor's Task tool, so it did not learn whether `subagent_type=Explore` fails when the listed type is `explore`.
- The decision-loss complaint cited for hit 3 is written about `grill-with-docs`, and that page's mitigation is `to-spec`. Hit 3 maps the same-conversation write onto `plan-as-artifact` because that is the spine's execution contract. Whether Key Decisions can hold every branch the maintainer would put in a spec is for the caller.
