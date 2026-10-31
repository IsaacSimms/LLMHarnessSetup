---
question: Which attached or configured capabilities, other than skill text and standing global instructions, would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine?
researched: 2026-10-30
confidence: partial
versions:
  - "Cursor docs (cursor.com/docs): fetched 2026-10-30"
  - "Claude Code docs (code.claude.com/docs): fetched 2026-10-30"
  - "Codex docs (learn.chatgpt.com/docs/agent-approvals-security): fetched 2026-10-30"
  - "Grok docs (docs.x.ai/build/features): fetched 2026-10-30"
  - "LLMHarnessSetup inventory (AGENTS.md, README.md, root skills, improve-codebase-architecture): read 2026-10-30"
---

# Which attached or configured capabilities, other than skill text and standing global instructions, would remove a repeated failure, a repeated manual step, or a mishandled decision on the current spine?

## Answer

Four attachables clear the bar. The largest is a read-only session for grill and design: Cursor's default CLI session is Agent mode with full tools, and Ask mode is the documented mode that explores without editing files. A verifier subagent, a background research subagent, and an after-edit copy hook remove the other three costs: a check that rubber-stamps itself, research that fills the parent session, and the manual copy from this repo into the install trees.

## Findings

- `AGENTS.md` treats two gates as critical: a design conversation is not implement permission, and plan mode makes zero file changes. It also says to offer check-work after non-trivial implementation and not to auto-run it. — [T1] AGENTS.md (`AGENTS.md`)
- `grill-me` forbids implementation, refactoring, and plan, recap, or handoff files during the grill unless the user explicitly asks for that artifact. Closing the grill writes a settled log in chat and does not write a file unless asked. — [T1] grill-me.md (`grill-me.md`)
- `plan-as-artifact` says writing the plan is not permission to implement. After the save, the agent reports the path and stops unless the user explicitly says to implement or continue. — [T1] plan-as-artifact.md (`plan-as-artifact.md`)
- `check-work` tells the orchestrator to run the verifier prompt in a fresh subagent when one is available, so the verifier re-checks claims instead of rubber-stamping them, and to offer that pass rather than auto-run it. — [T1] check-work.md (`check-work.md`)
- `research` exists to keep source reading in a background subagent. An inline pass burns the calling session's context, which the skill calls the cost it exists to avoid. Wayfinder research fires several of those subagents in parallel. The subagent writes one findings file. — [T1] research.md (`research.md`)
- `README.md` says that after a root skill edit, the same file is copied to the Cursor, Claude, and Grok install trees, and that those copies stay identical. — [T1] README.md (`README.md`)
- `improve-codebase-architecture` already tells the agent to explore with a subagent. Cursor's Explore subagent is built in and used automatically; the docs say it does not need configuration. — [T1] improve-codebase-architecture/SKILL.md (`improve-codebase-architecture/SKILL.md`); [T2] Cursor Subagents (https://cursor.com/docs/subagents)
- Cursor CLI defaults to Agent mode, described as full access to all tools. Ask mode is "read-only exploration without making changes" and "provides answers without editing files." Plan mode is described as designing an approach before coding. The same page says a prompt such as "do not write any code" is how to keep a planning turn from editing files. — [T2] Cursor CLI (https://cursor.com/docs/cli/overview); [T2] Using Agent in CLI (https://cursor.com/docs/cli/using)
- Cursor's Plan Mode page says the agent creates a plan, plans are saved under the home directory by default, the user can edit that plan as markdown, and building is a separate "click to build" step. The page does not say the mode blocks file edits. — [T2] Plan Mode (https://cursor.com/docs/agent/plan-mode)
- Interactive Cursor CLI asks for approval before terminal commands. Print mode (`-p`) has full write access, including write and shell. `--force` and `--yolo` force-allow commands unless they are explicitly denied. — [T2] Using Agent in CLI (https://cursor.com/docs/cli/using); [T2] Parameters (https://cursor.com/docs/cli/reference/parameters)
- Cursor `preToolUse` can deny a tool call. Its published input is the tool name, tool input, cwd, and model fields. It does not include the composer mode. `sessionStart` receives `composer_mode` of `agent`, `ask`, or `edit`, and session creation is not blocked. The `stop` hook can set `followup_message`, which Cursor submits as the next user message. — [T2] Hooks (https://cursor.com/docs/hooks)
- Cursor `afterFileEdit` fires after the agent edits a file and includes the absolute `file_path`. Command hooks on that event run a script. The published matcher for the event is `Write`. — [T2] Hooks (https://cursor.com/docs/hooks)
- A Cursor custom subagent has its own context window. `is_background: true` returns immediately and leaves the parent unblocked. `readonly: true` blocks file edits and state-changing shell commands. The description is what the parent reads when deciding to delegate. The docs' verifier pattern is a separate subagent that runs tests and reports what was claimed but is incomplete or broken, aimed at the case where the agent marks work done when it is not. — [T2] Subagents (https://cursor.com/docs/subagents)
- `agent sandbox run` uses a workspace read/write policy by default, and its `--network` flag defaults to false. The session flag `--sandbox enabled|disabled` is documented as a command-execution and network control that persists. The parameters page does not state that the session flag uses the same network default as `sandbox run`. — [T2] Parameters (https://cursor.com/docs/cli/reference/parameters); [T2] Cursor CLI (https://cursor.com/docs/cli/overview)
- Claude Code plan mode reads and explores, writes a plan, and does not edit source. Edits stay blocked until the user approves the plan, except in interactive sessions where bypass permissions are available. Approving the plan exits plan mode and starts editing in the mode the approval option names. — [T2] Choose a permission mode (https://code.claude.com/docs/en/permission-modes)
- A Claude Code `PreToolUse` deny applies in every permission mode, including bypass. In plan mode, a hook `allow` does not approve a file edit. — [T2] Automate actions with hooks (https://code.claude.com/docs/en/hooks-guide)
- Codex defaults to network off and an OS sandbox whose writes stay in the workspace. Read-only sandbox mode is the documented switch for chatting or planning without making changes (`/permissions`, or `sandbox_mode = "read-only"`). — [T2] Agent approvals & security (https://learn.chatgpt.com/docs/agent-approvals-security)
- Grok plan mode keeps edit tools limited while planning, and that limit is independent of auto and always-approve. Grok's built-in `plan` subagent drafts a plan with no shell and no edits. Its `explore` subagent is read, list, and search only. Subagents are on by default. — [T2] Permissions (https://docs.x.ai/build/features/permissions); [T2] Subagents (https://docs.x.ai/build/features/subagents)

## Recommendation

- **1. Read-only session for grill and design.**
- **Cost removed.** The agent edits the repo during `grill-me`, or during a design conversation, or treats a plan as permission to implement. That is the gate in `AGENTS.md` and the stop rule in `plan-as-artifact`.
- **Change.** On Cursor, start those sessions in Ask mode (`/ask` or `--mode=ask`). The CLI docs describe that mode as read-only and as answering without editing files. Leave it when the user explicitly asks for a file, including the plan artifact, or explicitly says to implement. Cursor Plan mode is the product's plan-then-build switch, and this pass did not find a write block on that page. On Claude Code, plan permission mode blocks source edits until the user approves a plan; that approval also leaves plan mode and starts editing, so it is a separate decision from this harness's implement instruction. On Codex, the read-only sandbox is the documented switch for planning without changes. On Grok, plan mode keeps edit tools limited, and the built-in plan subagent cannot edit.
- **Agent.** Cursor. Peers: Claude Code, Codex, and Grok.
- **Source.** [T2] Cursor CLI (https://cursor.com/docs/cli/overview), Using Agent in CLI (https://cursor.com/docs/cli/using), Claude Code permission modes (https://code.claude.com/docs/en/permission-modes), Codex agent approvals (https://learn.chatgpt.com/docs/agent-approvals-security), Grok permissions (https://docs.x.ai/build/features/permissions). [T1] `AGENTS.md`, `grill-me.md`, `plan-as-artifact.md`.

- **2. Verifier subagent.**
- **Cost removed.** `check-work` passes because the same session that wrote the code also judged it. The skill's remedy is a fresh verifier. Cursor documents the same failure: work marked done when the implementation is incomplete or broken.
- **Change.** Add the verifier subagent Cursor documents, at user scope so every project gets it, or at project scope. Its job is the check-work verifier prompt: run the project's real build and tests, and report what passed and what was only claimed. Invoke it when the user accepts the check-work offer. Leave `readonly` off, because that flag blocks state-changing shell commands and this pass must be able to run tests. A `stop` hook follow-up would submit the next user message on its own, which would auto-run the check the standing rule says to offer.
- **Agent.** Cursor.
- **Source.** [T2] Subagents (https://cursor.com/docs/subagents), Hooks (https://cursor.com/docs/hooks). [T1] `check-work.md`, `AGENTS.md`.

- **3. Background research subagent.**
- **Cost removed.** External-fact research and wayfinder research tickets run inline, so the parent session fills with source text. `research` calls that the failure the skill exists to avoid. The subagent still has to write the one findings file.
- **Change.** Add a Cursor research subagent whose description is the delegation hint for external-fact questions and wayfinder research tickets. Set it to run in the background so the parent keeps its window. Leave `readonly` off so it can write the findings file under `docs/research/` or the wayfinder research leaf. The preset isolates context. It does not, by itself, enforce the skill's production-endpoint and credential limits.
- **Agent.** Cursor. Grok subagents already return a summary from their own context; they are on by default.
- **Source.** [T2] Subagents (https://cursor.com/docs/subagents), Grok subagents (https://docs.x.ai/build/features/subagents). [T1] `research.md`.

- **4. After-edit copy into the install trees.**
- **Cost removed.** After a skill or `AGENTS.md` edit in this repo, someone copies that file into the Cursor, Claude, and Grok install paths by hand. A missed copy means the running agent follows a stale spine.
- **Change.** A Cursor `afterFileEdit` command hook sees the absolute path of each agent edit. When that path is a canonical harness file this repo already says to sync, the hook copies it to the install paths `README.md` names. The event does not see edits made outside the agent, so a hand edit of the harness still needs the same copy.
- **Agent.** Cursor.
- **Source.** [T2] Hooks (https://cursor.com/docs/hooks). [T1] README.md (`README.md`).

## Ruled out

- A Cursor rule that restates `AGENTS.md` belongs in `harness-prose-edits.md`. The CLI already loads `AGENTS.md` as rules.
- The Cursor docs' suggestion to prompt "do not write any code" during planning belongs in `harness-prose-edits.md`. It is another sentence of instruction.
- Cursor Plan mode as the enforcement of "zero file changes." The Plan Mode page saves a plan to the home directory and does not document a write block. Ask mode is the page that says files are not edited.
- Treating Claude Code's plan-approval button as this harness's implement permission. Approval exits plan mode and starts editing. The useful part is the block while the mode stays on, which is already inside rank 1.
- A `stop` hook that launches check-work. `followup_message` auto-submits the next user message, which collides with "offer check-work rather than auto-run."
- A global before-shell prompt hook that allows only read-only commands. Cursor documents that pattern. Applied to every session, it blocks the test commands `tdd` and `check-work` run.
- `permissions.json` terminal allowlists for `dotnet test` and peers. That removes an approval prompt. The standing rule wants explicit approval, and interactive CLI already asks before shell commands. This is a convenience, not a spine failure.
- Custom slash commands and Custom Mode that only pin an existing skill. The CLI already puts skills on the `/` menu. That is skill text staying loaded.
- Cursor's built-in Explore and Browser subagents, and Grok's built-in `explore` type. They are already on. `improve-codebase-architecture` already points at Explore.
- MCP servers, including auto-approving them with `--approve-mcps`. No server found in this pass maps to a spine step. Auto-approval widens a gate the research skill tells the agent to refuse.
- `--force` / `--yolo`, print mode's full write access, and Codex `danger-full-access`. They remove approval. They do not remove a spine failure.
- Session sandbox as a research production-endpoint ban. `sandbox run` documents network defaulting to false. The session `--sandbox` flag does not document that same default, so it does not clear the bar.
- Git worktrees (`--worktree`). Edits still happen. They land in another checkout. That does not stop an edit during grill or design.
- Model choice, billing, editor themes, and git hosting. Out of scope. Subagent docs discuss model IDs and token billing; those are not spine gates.
- Pi and anything under the retired tree. Not read.

## Unverified leads

None.

## Open questions

- Ask mode's docs say it does not edit files and does not make changes. They do not say whether an approved shell command, a delete, or an MCP write still changes the tree.
- `preToolUse` does not receive composer mode, and `sessionStart` cannot block. A hook that denies writes only during grill or plan is not supported by the schema read here. Mode can also change mid-session with Shift+Tab after `sessionStart` has fired.
- `readonly: true` blocks every file edit. The research skill requires one findings write. The docs read here do not show a single allowed output path.
- Grok's "edit tools stay limited" does not say whether a `docs/artifacts/` plan file is still writable, or whether every write is denied.
- `grill-me` requires a structured question tool (`AskQuestion`, or a host equivalent with a free-text box). The Cursor CLI pages read here do not document that tool. No attachable that adds it turned up.
- Whether `agent --sandbox enabled` uses the network-off policy documented for `agent sandbox run`.
