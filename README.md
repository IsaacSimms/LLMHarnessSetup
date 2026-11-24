# agent harness

Personal global instructions, skills, and subagents for a CLI coding agent.  
This repo is the source of truth I keep up to date and copy into `~/.claude/` on each machine.

---

## What’s in here

| Piece | Role |
|--------|------|
| **Global instructions** (`CLAUDE.md`) | Standing rules: coding style, approvals, plan mode, TDD, ubiquitous language (depth, seams, modules), workflow. Installed as `~/.claude/CLAUDE.md`. |
| **Skills** (`<skill>.md` at the repo root, plus `improve-codebase-architecture/`) | On-demand playbooks, loaded from `~/.claude/skills/<skill>/SKILL.md`. |
| **Subagents** (`Subagents/*.md`) | `researcher`, `verifier-check`, `content-conflict-resolver`, `readability-verifier`, and `tdd`. Loaded from `~/.claude/agents/`. |
| **IDE setups** (`IDE setups/vscode`; Cursor config is in `deprecated/cursor`) | User settings, keybindings, and extension ids for a new machine. See `IDE setups/README.md`. |

Agents should **not** invent parallel processes when a skill already covers the job.

---

## Install / sync

The layout, filenames, frontmatter, and tool names in this repo use **Claude Code** as the reference example.

### Claude Code (reference)

Copy (or symlink) into `~/.claude/`:

```text
CLAUDE.md                      →  ~/.claude/CLAUDE.md
<skill>.md                     →  ~/.claude/skills/<skill>/SKILL.md
improve-codebase-architecture/ →  ~/.claude/skills/improve-codebase-architecture/
Subagents/*.md                 →  ~/.claude/agents/*.md
```

**Policy:** Repo `CLAUDE.md` and `~/.claude/CLAUDE.md` stay **fully identical**. Installed skills stay **fully identical** to the repo. After editing a root `<skill>.md`, copy it to `~/.claude/skills/<skill>/SKILL.md`. After editing `improve-codebase-architecture/`, copy that folder to `~/.claude/skills/improve-codebase-architecture/`. Subagents load from `~/.claude/agents/`, not from this repo's `Subagents/` folder. After editing a file there, copy it to `~/.claude/agents/`. `~/.claude/skills/synced/` is managed by the app. Leave it alone.

### Installing for another harness

When installing or syncing this repo into a different coding agent or toolset (for example Antigravity/`agy`, Codex, Cursor, etc.), **prompt a model to adapt what is required to match the target harness's architecture** rather than copying files verbatim. Specifically, have it translate:

- **Install locations & layout:** global instructions file, skills directory, and subagent directory or discovery manifests (e.g. `~/.gemini/config/` for `agy`).
- **Tool names & schemas:** interactive question picker (`AskUserQuestion`), subagent invocation (`Agent` / `subagent_type`), and built-in explorer subagent names (`Explore`) to whatever the target harness exposes.
- **Frontmatter & instruction-file lists:** harness-specific YAML keys on skills/subagents (`background`, `disable-model-invocation`, `argument-hint`, etc.) and project instruction filenames referenced in skills (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.agents/rules/`).
- **Execution semantics & collisions:** foreground vs. async subagent ordering and any overlap with the target harness's built-in slash commands or subagent names.

---

## Workflow (default spine)

```text
create-docs-folder  →  docs/{artifacts,recaps,handoffs,checks,research,misc}
grill-me  →  plan-as-artifact  →  implement (tdd)  →  check-work
                ↓                      ↓
           docs/artifacts/        offer verify
                                         ↓
                    handoff (continue) / recap (done)

grill-me close may instead be implement directly (no plan file).
research is a session gate: external facts → researcher; this session decides.
```

`create-docs-folder` is optional and user-invoked. The later skills still create their own leaf when they write.

| Step | Skill | When |
|------|--------|------|
| Scaffold the docs tree | `create-docs-folder` | User asks for a docs folder for a project or directory |
| Stress-test decisions | `grill-me` | Design forks, multi-branch choices |
| Implement from the grill | `grill-me` | User picks implement directly. That is permission to implement in this thread. No plan file. |
| Multi-session fog | `wayfinder` | Too big for one session; decision map under `docs/artifacts/maps/<map-slug>/` |
| Execution contract | `plan-as-artifact` | Settled work, unless the grill close was implement directly |
| Build | `tdd` | Non-trivial features/fixes |
| Verify | `check-work` | After a non-trivial implement slice (offer; don’t auto-run). Delegates to `verifier-check`. |
| Continue elsewhere | `thread-handoff` | Forward-looking session transfer |
| Close the book | `recap-thread` | Backward-looking durable record |
| External facts | `research` | Session gate. `researcher` reads and recommends. This session decides. |

**Approvals:** A preference or design discussion is **not** permission to implement. Plan mode does not change code, and does not create, edit, or delete any file the user did not ask for. A skill the user just invoked may write only its own doc.

---

## Project docs layout

When skills write durable markdown into a **project** repo (not this harness repo), they use:

| Path | Skill |
|------|--------|
| `docs/artifacts/` | `plan-as-artifact` (flat `{slug}.md`) |
| `docs/artifacts/maps/<map-slug>/` | `wayfinder` |
| `docs/recaps/` | `recap-thread` |
| `docs/handoffs/` | `thread-handoff` |
| `docs/checks/` | `check-work` (optional save; force-save after failed fix loops) |
| `docs/research/` | `research` (standalone notes; wayfinder research stays under the map) |
| `docs/misc/` | No convention |

`create-docs-folder` creates the six top-level leaves (`artifacts`, `recaps`, `handoffs`, `checks`, `research`, `misc`), with a `.gitkeep` in each empty leaf. It does not create `plans/` or `artifacts/maps/`. `wayfinder` creates `docs/artifacts/maps/<map-slug>/` when it writes a map. A map already at `docs/plans/<map-slug>/` stays there. It does not write a README under `docs/`, does not create or edit any `.gitignore`, and does not create a file-type directory such as `screenshots/`. Screenshot files stay wherever they apply. A later run adds any standard leaf that is missing and leaves every other file where it is. `plans/` is not a standard leaf.

**Path resolution (docs-writing skills):**

1. If `docs/` exists in cwd → write under the skill subfolder (create if needed). No ask.
2. Else walk up for `docs/` and/or git root → **ask** with defaults.
3. Else ask; default create `./docs/<subfolder>/`.

**Path resolution (`create-docs-folder`):**

1. The user names a directory. That path is the project. Create or amend `docs/` there. Do not walk up, and do not ask when a parent `docs/` already exists. A missing path is created. A missing path whose last segment is `docs` is the docs root. A directory that already is the docs tree is amended in place.
2. No path. Use the docs-writing walk above, and scaffold the whole tree at the resolved `docs/` root.

Filenames:

- Plans: stable `{project?-}{feature-slug}.md`
- Recaps / handoffs / checks: `YYYY-MM-DD-topic-slug.md`

---

## Skills catalog

| Skill | Purpose |
|--------|---------|
| `create-docs-folder` | Scaffold `docs/` (`artifacts`, `recaps`, `handoffs`, `checks`, `research`, `misc`) |
| `grill-me` | One-at-a-time decision interview; settled log; options + recommend |
| `plan-as-artifact` | Structured plan → `docs/artifacts/` (frontmatter, phases, progress, cold-start) |
| `wayfinder` | Multi-session decision maps under `docs/artifacts/maps/<map-slug>/` |
| `tdd` | Red-green-refactor; public interface; vertical slices; scripting notes |
| `check-work` | Verify slice via `verifier-check`: claimed behavior, build/tests; optional `docs/checks/` |
| `thread-handoff` | Forward-looking handoff → `docs/handoffs/` |
| `recap-thread` | Backward-looking recap → `docs/recaps/` |
| `build-context-doc` | Generate/update project `Context.md` |
| `improve-codebase-architecture` | Deepening / interface design opportunities |
| `research` | Research via `researcher`: one findings file; parent decides |
| `learn` | Teach tightly-scoped skill |

Skills that ship with the app or arrive through `~/.claude/skills/synced/` are not required in this repo.

---

## Global instructions (highlights)

The full rules live in `CLAUDE.md`. Non-negotiables:

- **Plan mode:** no code changes, and no file the user did not ask for. A skill just invoked may write only its own doc. Implementation waits for an explicit instruction.  
- **TDD** for non-trivial work (use the `tdd` skill)  
- **Offer check-work** after non-trivial implement; it delegates to `verifier-check`; done = PASS or user waive  
- **Edit existing code first**; new files only when required. A skill the user just invoked may write only its own doc.  
- **Ubiquitous language:** Module, Interface, Implementation, Depth, Seam, Adapter, Leverage, Locality, Scan — mark `(UL)` when using these terms  
- **Pushback** when wrong; no empty affirmations; concise  
- **Comments:** title blocks + useful comments on sizable/unique code; don’t narrate the obvious  
- **Stack defaults:** prefer project scripts; else `verifier-check` defaults: Go → `go build`/`go test`, Shell → bats, Node → `npm test`, Python → pytest  

---

## Design principles baked into the harness

- **Depth** is leverage at the interface, not “always integration tests.”  
- Tests and callers share the same **interface**; don’t mock past the **seam** under test.  
- One adapter ⇒ hypothetical seam; two adapters ⇒ real seam.  
- Skills are **deep modules**: small invocation surface, rich behavior.  

---

## Contributing to this repo (personal)

1. Edit skills / global instructions here first.  
2. Name tools exactly as the agent exposes them (`AskUserQuestion`, `Agent`). Questions with options go through `AskUserQuestion`: 1–4 questions per call, 2–4 options each.  
3. Prefer **additions** over rewrites of working text.  
4. Sync `CLAUDE.md` to `~/.claude/CLAUDE.md`, root skills and `improve-codebase-architecture/` to `~/.claude/skills/`, and `Subagents/` to `~/.claude/agents/`.  
5. If a skill writes under `docs/`, use the shared path-resolution rules above. Scaffolding the tree is `create-docs-folder`.  

---

## License

Private / personal use unless noted otherwise.
