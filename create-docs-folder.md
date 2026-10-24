---
name: create-docs-folder
description: >
  Scaffold a project's docs tree: artifacts, recaps, handoffs, checks, plans,
  research, and misc, with a .gitkeep in each empty leaf. Use when the user
  asks to create a docs folder, make a docs folder for a project or directory,
  scaffold docs, or names create-docs-folder. A directory path they give is the
  project. With no path, resolve docs/ the same way the docs-writing skills do.
  Do not use this skill to write a plan, recap, handoff, check, map, or research
  note, and do not use it to add a README, a gitignore rule, or a screenshots folder.
metadata:
  short-description: "Scaffold the project docs tree"
---

# create-docs-folder

Stand up the project `docs/` tree. This skill creates directories. It does not write plans, recaps, handoffs, checks, maps, or research notes.

`plan-as-artifact`, `recap-thread`, `thread-handoff`, `check-work`, `wayfinder`, and `research` keep their own output-path resolvers. They create their own leaf when they write. This skill is the one-shot scaffold, run when the user asks for it.

## Leaves

Always these seven, as direct children of the docs root:

| Leaf | Owner |
|------|--------|
| `artifacts/` | `plan-as-artifact` |
| `recaps/` | `recap-thread` |
| `handoffs/` | `thread-handoff` |
| `checks/` | `check-work` |
| `plans/` | `wayfinder` (`docs/plans/<map-slug>/`) |
| `research/` | `research` standalone notes (`docs/research/`) |
| `misc/` | No convention. Do not tell an agent to use it or to avoid it. |

No `screenshots/` directory, and no other directory whose purpose is a file type. Screenshot files stay wherever they apply.

No README anywhere under the docs root. `Context.md` stays at the repo root; `build-context-doc` writes that file.

Do not create or edit any `.gitignore`, including `docs/.gitignore`.

## Resolve the docs root

### Named directory

The user gave a path. That path is the project. Do not walk up. Do not ask because a parent `docs/` already exists. Relative paths are fine.

1. The path is a file. Stop and report that. Create nothing.
2. The path does not exist.
   - Its final segment is `docs`. Create that path. It is the docs root.
   - Otherwise create the path, then `{path}/docs`. That `docs` directory is the docs root.
3. The path exists and is a directory.
   - Its name is `docs`. It is the docs root. Do not create a nested `docs/`.
   - Its name is one of the seven leaves, and its parent is named `docs` or already contains another of those leaves. The parent is the docs root.
   - Two or more of the seven leaves are already direct children. This directory is the docs root.
   - Otherwise the docs root is `{path}/docs`. Create it when missing. Amend it when present.

### No path

Use the same walk as `plan-as-artifact`, `recap-thread`, `thread-handoff`, and `check-work`. The result is the docs root, not one leaf.

1. Start at the agent cwd.
2. `docs/` exists in cwd. Use it. Do not ask.
3. Otherwise walk up for a `docs/` directory and/or a git root (`.git`).
4. Either is found. Ask before creating. Offer:
   - Nearest parent `docs/`
   - Git root without `docs/` → `{git-root}/docs/`
   - Both exist and differ. List both. Recommend the nearest `docs/`.
5. The walk finds nothing. Ask. Default `{cwd}/docs/`.

Wait for the answer before creating anything.

## Write the tree

For each of the seven leaves:

- Create the directory when it is missing.
- When the directory has no entries, create `.gitkeep` if that file is absent.
- When the directory has any entry, do not add `.gitkeep`.
- Do not delete an existing `.gitkeep`, or any other file or directory.
- Do not move or rename anything.

A later run adds any standard leaf that is missing, including one the user removed.

## Report

State the docs root. List directories and `.gitkeep` files created. List standard leaves that were already present. A path the user named gets no confirmation prompt.
