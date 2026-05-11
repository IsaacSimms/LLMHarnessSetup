---
name: context-md
description: >
  Generate or update a Context.md architectural reference document for a codebase Claude
  has not seen before. Use this skill whenever the user asks to "document this repo",
  "create a context file", "build a project reference", "map out this codebase", or
  "generate a context.md". Also trigger when the user says things like "you don't know
  this repo yet, here's the code" or "start a new project and document it first". The
  output is a durable, structured markdown file that serves as ground truth for all
  future AI-assisted work on the project.
---

# Context.md Skill

Produces a structured `Context.md` architectural reference for an unfamiliar codebase.
The document is a living artifact — written once, maintained incrementally — that gives
any AI assistant (or human) enough signal to work on the project without re-exploring
the repo every session.

---

## Phase 1 — Discovery

Before writing anything, explore the repo. Work through these in order, skipping any
that don't apply.

### 1.1 Orient
```bash
# Get the top-level shape
find . -maxdepth 2 -not -path '*/.git/*' -not -path '*/node_modules/*' \
       -not -path '*/bin/*' -not -path '*/obj/*' | sort

# Spot solution/project files, package manifests, and config roots
ls *.sln *.csproj *.fsproj package.json Cargo.toml go.mod pyproject.toml 2>/dev/null
```

### 1.2 Read entry points and manifests
Read these files fully — they reveal the stack, dependencies, and runtime topology:
- `.sln`, `*.csproj` — .NET project graph and package refs
- `package.json` / `package-lock.json` — JS/TS stack
- `Dockerfile`, `docker-compose.yml` — service topology
- `appsettings.json`, `.env.example` — configuration shape
- `Program.cs` or equivalent app entry — DI registration, middleware pipeline
- `README.md` — stated purpose and dev commands

### 1.3 Map the layers
For each project/module/package, identify:
- **Purpose** — what does it own?
- **Dependencies** — what does it reference?
- **Key types** — models, interfaces, controllers, services, hooks, components

Use targeted reads rather than reading every file:
```bash
# Find all interfaces (C#)
grep -r "public interface " --include="*.cs" -l

# Find all controllers
find . -name "*Controller.cs"

# Find all React feature folders
find src/features -maxdepth 1 -type d
```

### 1.4 Identify seams and patterns
Look for:
- Interface/implementation pairs (dependency injection seams)
- Factory or strategy patterns
- Middleware pipeline ordering
- Configuration binding patterns
- Exception handling and HTTP status mapping
- Service lifetime decisions (singleton vs scoped vs transient)

### 1.5 Resolve unknowns before writing
If discovery leaves gaps, ask the user one focused question per gap:
- "I see `ICodeExecutionService` has two implementations — is the switch config-driven or runtime?"
- "The `ChallengeCatalog` looks static — is there a DB-backed version planned?"

Do not write the document while holding unresolved architectural questions.

---

## Phase 2 — Write the Document

Write `Context.md` at the repo root (or wherever the user specifies).

Use the section structure below. Include every section that has content.
Omit sections that genuinely don't apply (e.g., no frontend → omit Frontend Conventions).
Do not add placeholder text or "TBD" sections.

---

### Document Structure

````markdown
# {ProjectName} — Context & Architecture Reference

{One-paragraph description: what the system does, who uses it, core value proposition.}

---

## Stack

| Layer | Technology |
|-------|-----------|
| {layer} | {technology + version if known} |

---

## Project Structure

```
{directory tree, 2–3 levels deep, with one-line annotation per entry}
```

### {ProjectName}.{Layer} — {brief purpose}
- `{Subfolder}/` — {what lives here and why}
- ...

{Repeat per project/package/module}

---

## Architecture Patterns

### Layering & Seams

| Seam | Interface | Implementations |
|------|-----------|----------------|
| {concern} | `{IInterface}` | `{Impl1}`, `{Impl2}` |

### Service Lifetimes
- **Singleton:** {list with rationale}
- **Scoped:** {list with rationale}
- **Transient:** {list with rationale — omit if none}

### Middleware Pipeline *(if applicable — order matters)*
1. {Middleware}
2. {Middleware}
...

### Exception → HTTP Mapping *(if applicable)*
| Exception | Status |
|-----------|--------|
| `{ExceptionType}` | {code + reason} |

### {Other patterns} *(add sections for Rate Limiting, Auth, Caching, etc. as present)*

---

## API Reference *(if HTTP API exists)*

| Method | Route | Request | Response | Notes |
|--------|-------|---------|----------|-------|
| {verb} | `{path}` | `{RequestDto}` | `{ResponseDto}` | {status codes or notes} |

### Key DTOs
- **{DtoName}** — {key fields and purpose}

---

## Key Models

| Model | Key Fields |
|-------|-----------|
| `{ModelName}` | `{field1}`, `{field2}`, ... |

---

## {Feature} Architecture *(one section per complex subsystem)*

{Narrative description of the workflow, phases, or state machine.}

**{Concept}:**
```
{code block showing composition, formula, or sequence if helpful}
```

---

## Frontend Conventions *(omit if no frontend)*

### {Library} Patterns
- {pattern}: {rationale or example}

### Component Hierarchy *(for non-obvious trees)*
```
{ComponentName}
├── {Child}   — {purpose}
└── {Child}   — {purpose}
```

---

## Naming Conventions

### Backend
| Thing | Convention | Example |
|-------|-----------|---------|
| {thing} | {convention} | `{Example}` |

### Frontend *(omit if no frontend)*
| Thing | Convention | Example |
|-------|-----------|---------|

---

## Code Style

{Bullet list of concrete style rules observed in the codebase. Include comment block conventions, XML doc rules, indentation, and any project-specific patterns.}

---

## Testing Conventions

### {Framework} *(one subsection per test layer)*
- **Location:** {pattern}
- **Method naming:** {pattern + example}
- **Mocking:** {tool and approach}
- **Assertions:** {preferred assertion style}

---

## Dev Commands

```bash
# {What it does}
{command}
```
````

---

## Phase 3 — Quality Gates

Before presenting the file, verify:

- [ ] Every project/package in the repo appears in Project Structure
- [ ] Every interface in the codebase appears in Layering & Seams (or is explained as internal)
- [ ] No section contains vague prose like "handles business logic" without specifics
- [ ] Stack table includes actual versions where discoverable
- [ ] API table covers all routes (grep `[HttpGet]`, `[HttpPost]`, `app.MapGet`, etc.)
- [ ] Dev commands are copy-pasteable and tested against the discovered config
- [ ] No invented content — if something is uncertain, note it with `*(unconfirmed)*`

---

## Phase 4 — Handoff

Present the file. Then tell the user:

> "This document reflects the repo as of today. As the architecture evolves, keep it
> updated — especially the Seams table, API Reference, and any new subsystem sections.
> Future sessions can reference this file instead of re-exploring the codebase."

If the user maintains the file in a known location, note it for future skill invocations.

---

## Maintenance Mode

If the user asks to *update* an existing `Context.md` (not create from scratch):

1. Read the existing file fully.
2. Identify what has changed — new routes, new services, renamed models, new patterns.
3. Do targeted discovery only for the changed areas.
4. Produce a diff-style summary of what changed before rewriting, so the user can review.
5. Rewrite only the affected sections; preserve unchanged sections verbatim.

---

## Notes on Scope

**Do include:**
- Anything a future developer (or AI) would need to understand to work confidently on the project
- Decisions that aren't obvious from filenames alone (why singleton? why Haiku vs Sonnet?)
- Patterns that are enforced project-wide (naming, comment style, test structure)

**Do not include:**
- Implementation details that belong in inline code comments
- Tutorial-style explanations of the technologies used
- Speculative future plans unless explicitly confirmed by the user
- Content that duplicates what's already in a `README.md` the document links to
