agent harness

Personal global instructions and skills for **CLI coding agents** (Claude Code, Grok Build, and peers).  
This repo is the source of truth I keep up to date and copy into the agent config directories on each machine.

---

## What’s in here

| Piece | Role |
|--------|------|
| **Global instructions** (`CLAUDE.md` / mirrored as `GROK.md`) | Standing rules: coding style, approvals, plan mode, TDD, ubiquitous language (depth, seams, modules), workflow |
| **Skills** (`skills/<name>/SKILL.md`) | On-demand playbooks agents load for grill, plan, TDD, verify, handoff, recap, architecture, etc. |

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
```

**Policy:** `CLAUDE.md` and `GROK.md` stay **fully identical**. Shared skills stay **fully identical** across trees. After editing here, sync both installs so Claude and Grok don’t drift.

---

## Workflow (default spine)

```text
grill-me  →  plan-as-artifact  →  implement (tdd)  →  check-work
                ↓                      ↓
           docs/artifacts/        offer verify
                                         ↓
                    handoff (continue) / recap (done)
```

| Step | Skill | When |
|------|--------|------|
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

**Path resolution (all docs-writing skills):**

1. If `docs/` exists in cwd → write under the skill subfolder (create if needed). No ask.
2. Else walk up for `docs/` and/or git root → **ask** with defaults.
3. Else ask; default create `./docs/<subfolder>/`.

Filenames:

- Plans: stable `{project?-}{feature-slug}.md`
- Recaps / handoffs / checks: `YYYY-MM-DD-topic-slug.md`

---

## Skills catalog

| Skill | Purpose |
|--------|---------|
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
4. Sync to `~/.claude` and `~/.grok` (or equivalent) after merge.  
5. If a skill writes under `docs/`, use the shared path-resolution rules above.  

---

## License

Private / personal use unless noted otherwise.
