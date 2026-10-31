---
question: Which ways of working, different from this repo's current spine, would be a large help? Name each idea and why it might help.
researched: 2026-10-30
confidence: partial
versions:
  - LLMHarnessSetup inventory (AGENTS.md, README.md, root skills, improve-codebase-architecture/): working tree read 2026-10-30
  - github/spec-kit: main@7370e592f3cd9c06e5b698983957e227c75b30e2 (2026-10-30); docs site read the same day
  - mattpocock/skills: main@a496f5513d4a4cf889bc48e189e579f4b0bdf6aa (2026-10-29)
  - obra/superpowers: main@2ce8aa15d05bd57213e6f746f953aafb7ded7a8b (v7.0.1, 2026-10-21)
  - Cursor docs (Cloud Agents; background-agents help): retrieved 2026-10-30
  - Claude Code docs (best practices, projects, dynamic workflows, agent teams): retrieved 2026-10-30
  - OpenAI Codex docs (workflows): retrieved 2026-10-30
  - xAI Grok Build docs (modes and commands): retrieved 2026-10-30
---

# Which ways of working, different from this repo's current spine, would be a large help? Name each idea and why it might help.

## Answer

Five published sequences claim a large help, and each one is a different order of work from grill, a plan artifact, in-session TDD, and an offered check. The two broadest claims are an unattended run that finishes in a pull request while the person is away, and a specification that is carried through planning, implementation, and a converge loop. Three more change how a large build is reviewed, cross-checked, or spread across agents. The sources state those payoffs. This pass did not measure them on this harness, so the order below is the scope each source claims.

## Findings

- This repo's default sequence is grill, then a plan artifact, then TDD for non-trivial work, then an offered check, with wayfinder when the work does not fit one session. — [T1] AGENTS.md (`/home/isaacsimms/Desktop/MiscRepos/LLMHarnessSetup/AGENTS.md`); [T1] README.md (`/home/isaacsimms/Desktop/MiscRepos/LLMHarnessSetup/README.md`)
- Check-work is a gate offered after a non-trivial slice. It prefers a fresh verifier when one is available, and it does not auto-run. Wayfinder is one writer: two sessions do not work the same map at once. — [T1] check-work.md (`/home/isaacsimms/Desktop/MiscRepos/LLMHarnessSetup/check-work.md`); [T1] wayfinder.md (`/home/isaacsimms/Desktop/MiscRepos/LLMHarnessSetup/wayfinder.md`)
- Cursor's background agents, which it now calls Cloud Agents, take a task from start to finish without interruption, on a remote VM, so the laptop can be closed. They build, test, open a pull request, and attach videos, screenshots, and logs for review without a local checkout. Many can run in parallel, and the local machine does not have to stay online. — [T2] Cursor, "What are background agents?" (https://cursor.com/help/ai-features/background-agents); [T2] Cursor, "Cloud Agents" (https://cursor.com/docs/cloud-agent)
- Claude Code's unattended guidance is the same shape of work with a tighter sign-off: define done, give the run a way to check itself, limit what it may touch, cap turns or spend, and have a person approve the merge or anything else that cannot be taken back. Once started, the long middle of the task proceeds on its own. Claude Projects put that middle in cloud threads that keep going after the laptop closes, in parallel, and the overview shows which threads finished and which pull requests are ready. — [T2] Claude Code, "Best practices" (https://code.claude.com/docs/en/best-practices); [T2] Claude Code, "Projects" (https://code.claude.com/docs/en/claude-projects)
- Codex documents a local plan, then a cloud chat that carries the plan and implements a milestone in an isolated environment, including in parallel with other work. The person reviews the cloud diff and opens the pull request from the cloud or pulls the branch back. — [T2] OpenAI Codex, "Workflows" (https://developers.openai.com/codex/workflows)
- Spec Kit's feature process is a constitution once per project, then specify, plan, tasks, implement, and converge, one step at a time, reviewing each artifact before the next. Implement and converge repeat until convergence reports `Converged`. The concept page says specifications become executable and generate the implementation, with multi-step refinement in place of one-shot generation from a prompt, and with the "what" set before the "how." — [T2] Spec Kit README (https://github.com/github/spec-kit/blob/7370e592f3cd9c06e5b698983957e227c75b30e2/README.md); [T2] Spec Kit, "What is Spec-Driven Development?" (https://github.github.io/spec-kit/concepts/sdd.html)
- Superpowers' subagent-driven development executes an existing plan by giving each task a fresh implementer, then a task review that must pass both spec compliance and code quality, then a whole-branch review. It tells the controller not to pause for the person between tasks. Conflicts are rulings recorded in a ledger. The stated reasons are isolated context, high quality with fast iteration, and the cost of a session parked on a question. Implementation subagents are not dispatched in parallel. — [T1] obra/superpowers `skills/subagent-driven-development/SKILL.md` (https://github.com/obra/superpowers/blob/2ce8aa15d05bd57213e6f746f953aafb7ded7a8b/skills/subagent-driven-development/SKILL.md)
- Claude Code dynamic workflows put the plan in a script the runtime executes. The script can run dozens to hundreds of subagents, keep intermediate results out of the conversation, and have independent agents adversarially review each other's findings, or draft a plan from several angles, before a result is reported. The docs aim that at jobs one conversation cannot coordinate: a codebase-wide audit, a large migration, or cross-checked research. The session stays responsive, and the script can be rerun. — [T2] Claude Code, "Dynamic workflows" (https://code.claude.com/docs/en/workflows)
- Grok Build workflows fan a task out across subagents in the background, verify, and return one result. The person describes the workflow; Grok authors it, smoke-checks it, and can save it. Workflows are on by default. — [T2] xAI Grok Build, "Modes and Commands" (https://docs.x.ai/build/modes-and-commands)
- `implement-spec` treats a spec's tickets as a blocking graph. A fresh implementer subagent, in its own worktree, builds each ready ticket with the `tdd` skill. A merger subagent lands that work on one integration branch, which opens the next frontier. Implementers run in the background for concurrency. Communication is context pointers. One `code-review` covers the integration branch at the end. — [T1] mattpocock/skills `skills/engineering/implement-spec/SKILL.md` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/implement-spec/SKILL.md)

## Recommendation

Ordered by how much of the work the source says the loop takes over. These are recommendations for a later review. They are not a decision to adopt one, and they contain no skill body.

- **1. Unattended start-to-finish run.**
- **Why it might help.** The build, the tests, and the pull request happen after the person leaves. Cursor's help doc says the agent takes the task from start to finish, the laptop can be closed, and the pull request carries video, screenshots, and logs so the result can be judged without checking out the branch. Claude's best-practices page keeps three decisions with the person: what done means, the bounds of the run, and approval of the merge. Codex uses the same handoff for a long milestone after a local plan. Parallel runs are part of the claim on all three.
- **Agent.** shared
- **Source.** [T2] Cursor, "What are background agents?" (https://cursor.com/help/ai-features/background-agents)

- **2. Spec-driven development through convergence.**
- **Why it might help.** A feature starts as a specification of what and why, then a technical plan, then dependency-ordered tasks, then implementation. Converge checks the code against those artifacts and the loop repeats until it reports `Converged`. Spec Kit's concept page says the specification generates the implementation through multi-step refinement, with each phase reviewed before the next command. That is a standing contract for the whole feature, carried past the moment implementation starts.
- **Agent.** shared
- **Source.** [T2] Spec Kit README, "Spec-Driven Development" (https://github.com/github/spec-kit/blob/7370e592f3cd9c06e5b698983957e227c75b30e2/README.md); [T2] Spec Kit, "What is Spec-Driven Development?" (https://github.github.io/spec-kit/concepts/sdd.html)

- **3. Per-task review gauntlet.**
- **Why it might help.** Each task gets a fresh implementer and then a separate review for spec compliance and for code quality. The next task waits until that review is clean, or until findings are parked with a recorded ruling at the fix-loop cap. A final review covers the branch. The skill's reason is quality with fast iteration: isolated context per task, reviews as the brakes, and a session that keeps moving because a parked question costs the day. The controller stops for irreversible, security-sensitive, or externally visible side effects, and when every path forward is a guess.
- **Agent.** shared
- **Source.** [T1] obra/superpowers `skills/subagent-driven-development/SKILL.md` (https://github.com/obra/superpowers/blob/2ce8aa15d05bd57213e6f746f953aafb7ded7a8b/skills/subagent-driven-development/SKILL.md)

- **4. Script-held fan-out with adversarial checks.**
- **Why it might help.** The plan lives in a script, so one conversation does not have to hold every intermediate result or decide the next worker turn by turn. Claude's docs say the script can run dozens to hundreds of agents and can require independent agents to challenge each other's findings before anything is reported, which it presents as more trustworthy than a single pass. The jobs named are a codebase-wide audit, a large migration, and cross-checked research. Grok's workflows are the same shape: fan out, verify, return one background result, with the session left free.
- **Agent.** Claude Code, Grok
- **Source.** [T2] Claude Code, "Dynamic workflows" (https://code.claude.com/docs/en/workflows); [T2] xAI Grok Build, "Modes and Commands," Workflows (https://docs.x.ai/build/modes-and-commands)

- **5. Parallel ticket-graph implementation.**
- **Why it might help.** A spec's tickets are a graph with blocking edges, so the ready frontier can be built at the same time. Each ticket gets a fresh implementer, its own worktree, and a TDD pass. Merges land on one integration branch through a merger subagent, which is what opens the next tickets. One code review covers the branch after the frontier is empty. The skill's reason is concurrency with a clean context per ticket, and a single branch that holds the whole spec.
- **Agent.** shared
- **Source.** [T1] mattpocock/skills `skills/engineering/implement-spec/SKILL.md` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/implement-spec/SKILL.md)

## Ruled out

- **Throwaway prototype for one ungrillable question.** The prototype skill builds throwaway code with no tests to answer a state-model or UI question, then sends the one-line answer back and leaves the code on a `prototype/` branch. The engineering note says to stop a grill that is growing by guessing. It also says to implement, per ticket, once the design is already settled, and it warns against prototyping the whole application. That is a detour on grill or wayfinder, so the skill belongs in `missing-spine-skills.md`. — [T1] mattpocock/skills `skills/engineering/prototype/SKILL.md` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/skills/engineering/prototype/SKILL.md); [T2] `docs/engineering/prototype.md` (https://github.com/mattpocock/skills/blob/a496f5513d4a4cf889bc48e189e579f4b0bdf6aa/docs/engineering/prototype.md)
- **Spec Kit idea assessment** (intake, research, define, shape, decide, with a `go`, `needs-clarification`, or `kill` verdict, and no source edits). Read in full. It is a real front door before specifying, and the guide calls a recorded kill a successful outcome. It sits below the five above because the claimed help is a verdict before a build, smaller than loops that carry, review, or fan out the build. The stage skills, if added beside grill, belong in `missing-spine-skills.md`. — [T2] Spec Kit, "Idea Assessment Quickstart" (https://github.github.io/spec-kit/guides/assessment.html)
- **Spec Kit bug fixing** (assess, fix, test, with a `verified` / `partial` / `failed` verdict). A bug path beside TDD and the absent `diagnose` name. Destination: `missing-spine-skills.md`. — [T2] Spec Kit README, "Bug fixing" (https://github.com/github/spec-kit/blob/7370e592f3cd9c06e5b698983957e227c75b30e2/README.md)
- **`code-review` as its own playbook.** `check-work.md` already routes ambitious structural review to a separate `code-review` skill, and that file is not in the inventory. `implement-spec` calls the same name at the end of the graph. The skill body was not read. Destination: `missing-spine-skills.md`. — [T1] check-work.md (`/home/isaacsimms/Desktop/MiscRepos/LLMHarnessSetup/check-work.md`)
- **Grok `/goal`.** An objective runs across many rounds. A panel of reviewer subagents has to agree, against evidence, before the goal is complete. Gaps send the work back. That completion rule is large, and the docs present it as the `/goal` command. Destination: `attached-capabilities.md`. — [T2] xAI Grok Build, "Modes and Commands," Goals (https://docs.x.ai/build/modes-and-commands)
- **Runners, flags, and triggers for loops already named above.** Cursor Automations, Slack or GitHub `@cursor`, cloud environment setup, cloud hooks, and cloud MCP; Claude `/loop`, `/schedule`, `/deep-research`, and the workflow script runtime; Grok `/loop`, `/create-workflow`, `/workflow`, and saved `.rhai` files; Spec Kit's `specify` CLI, extensions, hooks, and presets. The sequences are ranked here. The switches and saved runners belong in `attached-capabilities.md`. — [T2] Cursor, "Cloud Agents" (https://cursor.com/docs/cloud-agent); [T2] Claude Code best practices and dynamic workflows (URLs above); [T2] Grok Build modes and commands (URL above); [T2] Spec Kit README capabilities table (URL above)
- **Claude Code agent teams.** Teammates share a task list, message each other, and the docs single out research, review, new modules, and competing-hypothesis debugging. They are experimental, off until `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, and the same page says a single session costs less for routine work and fits sequential or same-file edits better. The cross-check help is already in the fan-out hit, and the parallel-build help is already in the ticket-graph hit. The enable flag, if tracked, belongs in `attached-capabilities.md`. — [T2] Claude Code, "Agent teams" (https://code.claude.com/docs/en/agent-teams)
- **A read-only reviewer subagent file** under Claude's `.claude/agents/`. That is a subagent preset. Destination: `attached-capabilities.md`. — [T2] Claude Code, "Best practices" (https://code.claude.com/docs/en/best-practices)
- **Codex "iterate on UI with live updates."** A tight styling loop against a running dev server. The page presents it as prompting practice. Smaller than the five ranked loops, and it does not describe a test-free prototype as the build path. — [T2] OpenAI Codex, "Workflows" (https://developers.openai.com/codex/workflows)
- **Grok plan mode.** Only the session plan file is editable until approval. That matches this repo's plan-mode fence closely enough that it is the same way of working. A behavior change to the fence would belong in `harness-prose-edits.md`. None is recommended here. — [T2] xAI Grok Build, "Modes and Commands," Plan (https://docs.x.ai/build/modes-and-commands); [T1] AGENTS.md
- **Parallel interface sketches.** Already in `improve-codebase-architecture/INTERFACE-DESIGN.md`: several subagents propose different interfaces for one deepening candidate. Already on the spine.
- **Codex `AGENTS.md` layering.** Instruction discovery for a file this repo already keeps. A product loading rule, not a new sequence.
- **Pages that restated a hit and added no further sequence.** Cursor's Cloud Agents reference restates the background-agent help page. Claude's best-practices page adds the unattended checklist and the cost notes used above, and no sixth loop.

## Unverified leads

None.

## Open questions

- The five sources do not rank themselves against each other. The order is the scope of help each one claims. A later review can reorder them from session pain this pass could not see.
- Unattended runs, the review gauntlet's rulings, and the fan-out scripts proceed through implementation without a person between steps. Claude's unattended page still wants a person on the merge. This note does not resolve that against the harness rule that a design conversation is not implement permission.
- Spec Kit's per-command skill templates were not opened. The process text is the README, the SDD concept page, and the assessment guide. How converge writes `tasks.md` was not taken from the persistence concept page.
- Superpowers' brainstorming, writing-plans, and inline executing-plans skills were not read. They may be a restatement of grill, plan, and a single-session implement.
- `to-spec`, `to-tickets`, `grill-with-docs`, and the body of `code-review` were not opened. Search excerpts named them in the same upstream route as `implement-spec`. Their owning file, if they are only missing playbooks, is `missing-spine-skills.md`.
- Grok `/goal` is a completion loop with a reviewer panel. It is listed under Ruled out because the docs present it as a slash command. If `attached-capabilities.md` rejects it as a sequence, it still needs an owner.
