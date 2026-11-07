# agent harness

Personal global instructions and skills for **Coding agents** (Cursor CLI, Claude Code, and peers).  
This repo is the source of truth I keep up to date and copy into the agent config directories on each machine.

---

## What’s in here

| Piece | Role |
|--------|------|
| **Global instructions** (`AGENTS.md`) | Standing rules: coding style, approvals, plan mode, TDD, ubiquitous language (depth, seams, modules), workflow. Cursor copy is `~/.cursor/AGENTS.md`. |
| **Skills** (`<skill>.md` at the repo root, plus `improve-codebase-architecture/`) | On-demand playbooks. Cursor CLI reads `~/.cursor/skills/<skill>/SKILL.md`. |
| **Subagents** (`Subagents/*.md`) | `researcher` and `verifier-check`. Cursor loads `~/.cursor/agents/`. |
| **IDE setups** (`IDE setups/vscode`, `IDE setups/cursor`) | User settings, keybindings, and extension ids for a new machine. See `IDE setups/README.md`. |

Agents should **not** invent parallel processes when a skill already covers the job.

---

## Install / sync

Copy (or symlink) into the agent’s user config, for example:

```text
# Cursor
AGENTS.md                      →  ~/.cursor/AGENTS.md
<skill>.md                     →  ~/.cursor/skills/<skill>/SKILL.md
improve-codebase-architecture/ →  ~/.cursor/skills/improve-codebase-architecture/
Subagents/*.md                 →  ~/.cursor/agents/*.md

# Claude Code
AGENTS.md  →  ~/.claude/CLAUDE.md
skills     →  ~/.claude/skills/

# Grok Build (peer)
AGENTS.md  →  ~/.grok/rules/GROK.md
skills     →  ~/.grok/skills/
```

**Policy:** Repo `AGENTS.md` and `~/.cursor/AGENTS.md` stay **fully identical**. Peer copies of that same text (`~/.claude/CLAUDE.md`, `~/.grok/rules/GROK.md`) stay identical when those installs exist. Shared skills stay **fully identical** across trees. After editing a root `<skill>.md`, copy it to `~/.cursor/skills/<skill>/SKILL.md` and to the Claude and Grok skill dirs. After editing `improve-codebase-architecture/`, copy that folder to `~/.cursor/skills/improve-codebase-architecture/` and to those same peer skill dirs. Cursor loads subagents from `~/.cursor/agents/`, not from this repo's `Subagents/` folder. After editing a file there, copy it to `~/.cursor/agents/`.

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

Platform-only skills (e.g. Grok `help`, `imagine`) may live only in a specific agent install and are not required in this repo.

---

## Global instructions (highlights)

The full rules live in `AGENTS.md`. Non-negotiables:

- **Plan mode:** no code changes, and no file the user did not ask for. A skill just invoked may write only its own doc. Implementation waits for an explicit instruction.  
- **TDD** for non-trivial work (use the `tdd` skill)  
- **Offer check-work** after non-trivial implement; it delegates to `verifier-check`; done = PASS or user waive  
- **Edit existing code first**; new files only when required. A skill the user just invoked may write only its own doc.  
- **Ubiquitous language:** Module, Interface, Implementation, Depth, Seam, Adapter, Leverage, Locality, Scan — mark `(UL)` when using these terms  
- **Pushback** when wrong; no empty affirmations; concise  
- **Comments:** title blocks + useful comments on sizable/unique code; don’t narrate the obvious  
- **Stack defaults:** prefer project scripts; else `verifier-check` defaults: .NET → `dotnet build`/`test`, PowerShell → Pester, Node → `npm test`, Python → pytest  

---

## Design principles baked into the harness

- **Depth** is leverage at the interface, not “always integration tests.”  
- Tests and callers share the same **interface**; don’t mock past the **seam** under test.  
- One adapter ⇒ hypothetical seam; two adapters ⇒ real seam.  
- Skills are **deep modules**: small invocation surface, rich behavior.  

---

## Contributing to this repo (personal)

1. Edit skills / global instructions here first.  
2. Keep skill bodies **agent-neutral** (Coding agents, not a single product name).  
3. Prefer **additions** over rewrites of working text.  
4. Sync `AGENTS.md` to `~/.cursor/AGENTS.md`. Sync root skills and `improve-codebase-architecture/` to `~/.cursor/skills`, and `Subagents/` to `~/.cursor/agents/`. Peer installs (`~/.claude`, `~/.grok`, or equivalent) get the same text.  
5. If a skill writes under `docs/`, use the shared path-resolution rules above. Scaffolding the tree is `create-docs-folder`.  

---

## License

Private / personal use unless noted otherwise.
