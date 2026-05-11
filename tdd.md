---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development. Also trigger when user asks how to write tests for a feature, wants to add test coverage, or says things like "let's TDD this" or "write tests first". When a user wants to implement any non-trivial feature or fix a bug with quality guarantees, suggest TDD as the approach and use this skill.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

Good tests are integration-style — they exercise real code paths through public APIs and describe *what* the system does, not *how*. A good test reads like a spec: `"user can checkout with valid cart"` tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

Bad tests are coupled to implementation: they mock internal collaborators, test private methods, or verify through external means (querying a DB directly instead of using the public interface). Warning sign: your test breaks when you refactor but behavior hasn't changed.

→ See `references/tests.md` for good/bad test examples  
→ See `references/mocking.md` for when and how to mock  
→ See `references/refactoring.md` for refactor candidates after GREEN

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
- **Get user approval** before proceeding

Ask: *"What should the public interface look like? Which behaviors are most critical to test?"*

### 2. Tracer Bullet

Write ONE test that confirms ONE behavior end-to-end:

```
RED:   Write test → confirm it fails for the right reason
GREEN: Write minimal code to pass → confirm it passes
```

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

### 4. Refactor

After all tests pass, look for refactor candidates (see `references/refactoring.md`):
- Extract duplication
- Deepen modules
- Apply SOLID where natural
- Run tests after **each** refactor step

**Never refactor while RED. Get to GREEN first.**

---

## Per-Cycle Checklist

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test only
[ ] No speculative features added
[ ] Test name says WHAT, not HOW
```

---

## Language Notes

Examples in the reference files use TypeScript. The same principles apply in C#/xUnit — substitute `[Fact]`/`[Theory]` for `test()`, `Assert.Equal` for `expect().toBe()`, and use constructor injection for DI at system boundaries.
