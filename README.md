# agent harness

Personal global instructions and skills for **CLI coding agents** (Claude Code, Grok Build, and peers).  
This repo is the source of truth I keep up to date and copy into the agent config directories on each machine.

---

## What’s in here

| Piece | Role |
|--------|------|
| **Global instructions** (`CLAUDE.md` / mirrored as `GROK.md`) | Standing rules: coding style, approvals, plan mode, TDD, ubiquitous language (depth, seams, modules), workflow |
| **Skills** (`<skill>.md` at the repo root) | On-demand playbooks. Cursor CLI reads the matching `~/.cursor/skills/<skill>/SKILL.md`. |

Agents should **not** invent parallel processes when a skill already covers the job.

---

## Install / sync

Copy (or symlink) into the agent’s user config, for example:

```text
# Claude Code
CLAUDE.md  →  ~/.claude/CLAUDE.md
skills/*   →  ~/.claude/skills/

# Grok Build (same content policy)
GROK.md    →  ~/.grok/rules/GROK.md
skills/*   →  ~/.grok/skills/

# Cursor CLI
<skill>.md →  ~/.cursor/skills/<skill>/SKILL.md
```

**Policy:** `CLAUDE.md` and `GROK.md` stay **fully identical**. Shared skills stay **fully identical** across trees. After editing a root `<skill>.md`, copy it to `~/.cursor/skills/<skill>/SKILL.md` and to the Claude and Grok installs.

---

## Workflow (default spine)

```text
create-docs-folder  →  docs/{artifacts,recaps,handoffs,checks,plans,research,misc}
grill-me  →  plan-as-artifact  →  implement (tdd)  →  check-work
                ↓                      ↓
           docs/artifacts/        offer verify
                                         ↓
                    handoff (continue) / recap (done)
```

`create-docs-folder` is optional and user-invoked. The later skills still create their own leaf when they write.

| Step | Skill | When |
|------|--------|------|
| Scaffold the docs tree | `create-docs-folder` | User asks for a docs folder for a project or directory |
| Stress-test decisions | `grill-me` | Design forks, multi-branch choices |
| Multi-session fog | `wayfinder` | Too big for one session; decision map under `docs/plans/` |
| Execution contract | `plan-as-artifact` | Settled work ready to implement |
| Build | `tdd` | Non-trivial features/fixes |
| Verify | `check-work` | After a non-trivial implement slice (offer; don’t auto-run) |
| Continue elsewhere | `thread-handoff` | Forward-looking session transfer |
| Close the book | `recap-thread` | Backward-looking durable record |

**Approvals:** A preference or design discussion is **not** permission to implement. Plan mode = zero file changes until an explicit implement instruction.

---

## Project docs layout

When skills write durable markdown into a **project** repo (not this harness repo), they use:

| Path | Skill |
|------|--------|
| `docs/artifacts/` | `plan-as-artifact` |
| `docs/recaps/` | `recap-thread` |
| `docs/handoffs/` | `thread-handoff` |
| `docs/checks/` | `check-work` (optional save; force-save after failed fix loops) |
| `docs/plans/<map-slug>/` | `wayfinder` |
| `docs/research/` | `research` (standalone notes; wayfinder research stays under the map) |
| `docs/misc/` | No convention |

`create-docs-folder` creates that tree in one shot: the seven leaves above, with a `.gitkeep` in each empty leaf. It does not write a README under `docs/`, does not create or edit any `.gitignore`, and does not create a file-type directory such as `screenshots/`. Screenshot files stay wherever they apply. A later run adds any standard leaf that is missing and leaves every other file where it is.

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
| `create-docs-folder` | Scaffold `docs/` (`artifacts`, `recaps`, `handoffs`, `checks`, `plans`, `research`, `misc`) |
| `grill-me` | One-at-a-time decision interview; settled log; options + recommend |
| `plan-as-artifact` | Structured plan → `docs/artifacts/` (frontmatter, phases, progress, cold-start) |
| `wayfinder` | Multi-session decision maps under `docs/plans/` |
| `tdd` | Red-green-refactor; public interface; vertical slices; scripting notes |
| `check-work` | Verify slice: build/tests, plan AC, short depth/UL pass; optional `docs/checks/` |
| `thread-handoff` | Forward-looking handoff → `docs/handoffs/` |
| `recap-thread` | Backward-looking recap → `docs/recaps/` |
| `build-context-doc` | Generate/update project `Context.md` |
| `improve-codebase-architecture` | Deepening / interface design opportunities |
| `research` | High-trust research with captured findings |
| `learn` | Teach tightly-scoped skill |
| `to-prd` | Conversation → PRD |
| `diagnose` | Diagnostic workflow |

Platform-only skills (e.g. Grok `help`, `imagine`) may live only in a specific agent install and are not required in this repo.

---

## Global instructions (highlights)

The full rules live in `CLAUDE.md`. Non-negotiables:

- **Plan mode:** zero file edits until explicit implement approval  
- **TDD** for non-trivial work (use the `tdd` skill)  
- **Offer check-work** after non-trivial implement; done = PASS or user waive  
- **Edit existing code first**; new files only when required  
- **Ubiquitous language:** Module, Interface, Implementation, Depth, Seam, Adapter, Leverage, Locality, Scan — mark `(UL)` when using these terms  
- **Pushback** when wrong; no empty affirmations; concise  
- **Comments:** title blocks + useful comments on sizable/unique code; don’t narrate the obvious  
- **Stack defaults:** prefer project scripts; else .NET → `dotnet build`/`test`, PowerShell → Pester when present  

---

## Design principles baked into the harness

- **Depth** is leverage at the interface, not “always integration tests.”  
- Tests and callers share the same **interface**; don’t mock past the **seam** under test.  
- One adapter ⇒ hypothetical seam; two adapters ⇒ real seam.  
- Skills are **deep modules**: small invocation surface, rich behavior.  

---

## Contributing to this repo (personal)

1. Edit skills / global instructions here first.  
2. Keep skill bodies **agent-neutral** (CLI coding agents, not a single product name).  
3. Prefer **additions** over rewrites of working text.  
4. Sync to `~/.claude`, `~/.grok`, and `~/.cursor/skills` (or equivalent) after merge.  
5. If a skill writes under `docs/`, use the shared path-resolution rules above. Scaffolding the tree is `create-docs-folder`.  

---

## License

Private / personal use unless noted otherwise.
