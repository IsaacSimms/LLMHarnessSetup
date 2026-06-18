# Global CLAUDE.md OR GROK.md

## Coding Conventions

When beginning a function, class, or important block of code, give it a title comment using the convention:
```
// == Title Here == //
```
Adapt the comment syntax to the language (e.g., `# == Title Here == #` for Python/Bash, `<!-- == Title Here == -->` for HTML/XML).

For multi-line or longer messages that aren't block titles, place the comment on the line above the code in question.

For single-line comments about a specific line of code, place the comment on the same line. Leave space between the code and the comment, and align inline comments with each other where practical. Alignment is a preference, not a strict rule.

### XML Summary Comments (`/// <summary>`)
Only use `/// <summary>` XML doc comments for class-level, interface-level, or file-level documentation (i.e., describing an entire class/interface/enum). Do NOT use `/// <summary>` on individual properties, fields, enum members, or single methods. For those, use a standard inline `//` comment on the same line as the code. Similarly, do not use `/// <inheritdoc />` — it adds no real value.

If a specific framework, architecture, service, programming langauge, or any entity related to the work that is currently being done has any naming conventions, best practices, guidelines, that are in place for best outcomes with that entity, follow those.

## General
When you are Overwriting a file, I want an explination of what change, why it changed, and how that will effect the codebase/project. I would prefer you use inline changes whenever possible.

I want you to do test driven devleopment whenever possible.

I am reviewing every line of code and I expect clean, elegant, best practices, professional work. 

Attempt to edit and refactor existing code whenever possible, only create new code or new files whenever absolutely required. This applies to when you are asked to edit, change, update, or create functionalities and codebases. This applies to when you are asked to debug.

## Interaction preferences

Expects direct pushback when wrong; don't cave to incorrect challenges
No affirmations ("you're right", "good thought", "definitely")
Concise and direct

## VS Debugger
When investigating bugs, unexpected behavior, or test failures in code open in Visual Studio, use the vs-debugger MCP tools to debug interactively rather than guessing. This works for any language VS supports (C#, C++, Python, TypeScript, etc.). Call `vs_status` first to confirm VS is connected, set breakpoints with `vs_set_breakpoint`, then `vs_launch` to start debugging. Use `vs_get_locals`, `vs_evaluate`, and `vs_get_callstack` to inspect state at breakpoints. VS must be running at the same elevation level as Claude Code.

## Approval & Plan Mode — CRITICAL

**Approvals to implement are explicit and unambiguous.** Do NOT interpret a stated preference, opinion, or design discussion as permission to make a change.

**In plan mode, make zero file changes. None. Ever.** Plan mode means produce a plan only. Do not create, edit, or delete any file — not even a "small" one — regardless of how confident you are. Wait for an explicit implementation instruction outside of plan mode.


## Ubiquitous Language
Shared vocabulary between the user and the LLM. Use these terms exactly — don't substitute "component," "service," "API," or "boundary." Consistent language is the whole point. All of the words listed below have thier own meaning outside of our Ubiqitous Language as well. If you are pulling from this list as you think or talk to the user, put (UL) next to the output. 

**Module**
Anything with an interface and an implementation. Deliberately scale-agnostic — applies equally to a function, class, package, or tier-spanning slice.


**Interface**
Everything a caller must know to use the module correctly. Includes the type signature, but also invariants, ordering constraints, error modes, required configuration, and performance characteristics.


**Implementation**
What's inside a module — its body of code. Distinct from **Adapter**: a thing can be a small adapter with a large implementation (a Postgres repo) or a large adapter with a small implementation (an in-memory fake). Reach for "adapter" when the seam is the topic; "implementation" otherwise.

**Depth**
Leverage at the interface — the amount of behaviour a caller (or test) can exercise per unit of interface they have to learn. A module is **deep** when a large amount of behaviour sits behind a small interface. A module is **shallow** when the interface is nearly as complex as the implementation.

**Seam** _(from Michael Feathers)_
A place where you can alter behaviour without editing in that place. The *location* at which a module's interface lives. Choosing where to put the seam is its own design decision, distinct from what goes behind it.


**Adapter**
A concrete thing that satisfies an interface at a seam. Describes *role* (what slot it fills), not substance (what's inside).

**Leverage**
What callers get from depth. More capability per unit of interface they have to learn. One implementation pays back across N call sites and M tests.

**Locality**
What maintainers get from depth. Change, bugs, knowledge, and verification concentrate at one place rather than spreading across callers. Fix once, fixed everywhere.

**Scan**
Delibrately and intentionally analyze the file, codebase, or resource in its entirety. Ask user to define scope if needed. 

## Principles

- **Depth is a property of the interface, not the implementation.** A deep module can be internally composed of small, mockable, swappable parts — they just aren't part of the interface. A module can have **internal seams** (private to its implementation, used by its own tests) as well as the **external seam** at its interface.
- **The deletion test.** Imagine deleting the module. If complexity vanishes, the module wasn't hiding anything (it was a pass-through). If complexity reappears across N callers, the module was earning its keep.
- **The interface is the test surface.** Callers and tests cross the same seam. If you want to test *past* the interface, the module is probably the wrong shape.
- **One adapter means a hypothetical seam. Two adapters means a real one.** Don't introduce a seam unless something actually varies across it.

## Relationships

- A **Module** has exactly one **Interface** (the surface it presents to callers and tests).
- **Depth** is a property of a **Module**, measured against its **Interface**.
- A **Seam** is where a **Module**'s **Interface** lives.
- An **Adapter** sits at a **Seam** and satisfies the **Interface**.
- **Depth** produces **Leverage** for callers and **Locality** for maintainers.

## Rejected framings

- **Depth as ratio of implementation-lines to interface-lines** (Ousterhout): rewards padding the implementation. We use depth-as-leverage instead.
- **"Interface" as the TypeScript `interface` keyword or a class's public methods**: too narrow — interface here includes every fact a caller must know.
- **"Boundary"**: overloaded with DDD's bounded context. Say **seam** or **interface**.
