---
name: tdd
description: >
  Test-driven development with red-green-refactor loop. Use when user wants to build features or fix
  bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first
  development. Also trigger when user asks how to write tests for a feature, wants to add test coverage,
  or says things like "let's TDD this" or "write tests first". When a user wants to implement any
  non-trivial feature or fix a bug with quality guarantees, suggest TDD as the approach and use this skill.
metadata:
  short-description: "Red-green-refactor test-driven development"
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

Good tests exercise real behavior at the **Module**'s public **Interface** (UL) — the surface callers and tests share. They describe *what* the system does, not *how*. A ood test reads like a spec: `"user can checkout with valid cart"` tells you exactly what capability exists. Prefer fewer, deeper tests over many shallow ones. These tests survive refactors because they don't care about internal structure.

A pure function with no collaborators can still be a correct unit test if it only crosses that interface. "Deep" here means leverage at the interface, not "always cross process/DB/network."

Bad tests are coupled to implementation: they mock past the **Seam** (UL) under test, test private methods, or verify through external means (querying a DB directly instead of using the public interface). Warning sign: your test breaks when you refactor but behavior hasn't changed.

**Fakes/mocks are valid only at a real seam** — where more than one **Adapter** (UL) exists or the collaborator is truly external (network, clock, OS). One adapter means a hypothetical seam; don't invent doubles for internals.

---

## Anti-Pattern: Horizontal Slices

**DO NOT** write all tests first, then all implementation.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical tracer bullets):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
```

Horizontal slicing produces tests that verify imagined behavior and are insensitive to real changes. Vertical slices let each test respond to what you learned from the previous cycle.

---

## Workflow

### 1. Plan (before writing any code)

- Confirm the **public interface** shape with the user
- Confirm **which behaviors** to test — not all of them, the ones that matter
- Identify deep module opportunities (small interface, rich behavior)
- List behaviors as test names (not implementation steps)
- **Get user approval** on interface shape + critical behaviors before proceeding (not every subsequent red-green cycle)

Ask: *"What should the public interface look like? Which behaviors are most critical to test?"*

### 2. Tracer Bullet

Write ONE test that confirms ONE behavior end-to-end:

```
RED:   Write test → confirm it fails for the right reason
GREEN: Write minimal code to pass → confirm it passes
```

**Right reason**: the assertion fails because the behavior is missing — not because of a compile error, missing import, wrong path, or broken test harness. Fix plumbing first; only then is it a true RED.

This proves the path works before building out.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → it fails
GREEN: Minimal code to pass → it passes
```

Rules:
- One test at a time
- Only enough code to pass the current test
- Don't anticipate future tests
- Tests must use the public interface only
- Confirm each RED fails for the right reason before writing production code

### 3b. Bug fixes

Same loop, different entry:

```
RED:   Write a failing test that reproduces the bug (minimal, through the public interface)
GREEN: Minimal fix to make that test pass
REFACTOR: Only after green
```

Do not "just patch" then add a test afterward unless the user explicitly waives TDD.

### 4. Refactor

After all tests pass, look for refactor candidates:
- Extract duplication
- Deepen modules
- Apply SOLID where natural
- Run tests after **each** refactor step

**Never refactor while RED. Get to GREEN first.**
Internal structure is not sacred; the interface's tests staying green is.

---

## Per-Cycle Checklist

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] RED failed for missing behavior (not plumbing)
[ ] Code is minimal for this test only
[ ] No speculative features added
[ ] Test name says WHAT, not HOW
```

---

## Language / Stack Notes

Principles are language-agnostic. Map the runner to the stack:

- **C#**: xUnit `[Fact]`/`[Theory]`, FluentAssertions, constructor injection at system boundaries
- **TypeScript/JS**: `test()`/`it()`, project assertion lib; same public-interface rules
- **Python**: pytest; test public functions/modules, not private `_helpers` unless that *is* the product

---

## Scripting Languages (PowerShell, Bash, Python CLIs, etc.)

TDD still applies; the public interface looks different.

- **Interface** = what callers observe: exported functions/cmdlets, parameters, return values/objects, stdout/stderr/exit codes — not private helpers or line-by-line control flow.
- Prefer vertical slices: one behavior (e.g. "rejects missing path with non-zero exit") → minimal implementation → next behavior.
- Side effects (files, registry, network): assert outcomes via temp fixtures, or fake only at real external seams. Do not mock every internal call.
- One-shot glue with no reuse: skip heavy TDD. If logic will grow, extract a testable module first and drive *that* with red-green-refactor.
- **PowerShell**: Pester. Same rules as xUnit — behavior names, public surface only.

Characterization note: for untested legacy scripts, pin current behavior with a failing-or-passing snapshot test first, then change — don't invent the ideal API in the dark.
