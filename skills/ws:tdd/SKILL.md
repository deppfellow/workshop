---
name: ws:tdd
description: >
  Test-driven development with red-green-refactor vertical slices.
  Consumes a single task from ws:plan. Tests prove behavior through public interfaces.
  Use when building features, fixing bugs test-first, or implementing a plan slice.
trigger: ws:tdd, tdd, test-first, red-green-refactor
---

# ws:tdd

Build behavior, not implementation. Red → Green → Refactor. One test. One slice. Vertical always.

Override `rules.md` guardrails. Exec permitted.

## Tone

Caveman-terse.

## Philosophy

**Tests verify behavior through public interfaces.** Code can change entirely; tests shouldn't. Good test reads like specification — "user can checkout with valid cart" tells what, not how.

**Tests survive refactors** because they don't care about internal structure. If test breaks when you refactor but behavior unchanged — test was testing implementation.

## Process

### Stage 1: SETUP

1. Read target slice from `docs/<slug>/plan.md`
2. Read `docs/<slug>/spec.md` for context (stories, architecture, test strategy)
3. Confirm interfaces with user:
   - What public interfaces change?
   - Which behaviors to test? (prioritize — can't test everything)
   - Identify test seams and deep modules
4. Identify opportunities for deep modules (small interface, deep implementation)

**Gate:** Confirm interface changes and test priorities with user before writing code.

**Dispatch `ws:workmate` investigator** for codebase exploration if needed.

### Stage 2: BUILD

Vertical slices. Never horizontal.

**Tracer bullet first:**

```text
RED:   Write ONE test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

**Incremental loop:**

```text
For each remaining behavior:
  RED:   Write next test → fails
  GREEN: Minimal code to pass → passes
```

**Rules:**

- One test at a time
- Only enough code to pass current test
- Never anticipate future tests
- Tests describe behavior, not implementation
- Tests use public interface only

**Dispatch `ws:workmate` builder** for surgical edits (1-2 files max).

### Stage 3: REFACTOR

Only when GREEN. Never red.

1. Extract duplication
2. Deepen modules (move complexity behind simple interfaces)
3. Apply SOLID where natural
4. Consider what new code reveals about existing code
5. Run tests after each refactor step

### Stage 4: VERIFY

1. All tests green
2. Regression: re-run full test suite for affected area
3. Check against plan acceptance criteria
4. Present summary to user

## Anti-Patterns

**DO NOT write all tests first, then all implementation** (horizontal slicing). This produces crap tests:

- Tests written in bulk test IMAGINED behavior, not ACTUAL behavior
- Tests become insensitive to real changes
- You outrun your headlights

**DO NOT:**

- Mock internal collaborators (test seams, not internals)
- Test private methods (test through public interface)
- Verify through external means (query DB directly instead of interface)
- Refactor while RED (get to GREEN first)

## Guardrails

- Never skip the setup confirmation gate
- One test at a time — no batch writing
- Tests must survive internal refactor
- `ws:workmate` builder: if change >2 files, reject — escalate to main agent

## Integration

- Consumes `ws:plan` slice
- Dispatches `ws:workmate` builder for surgical edits
- Dispatches `ws:workmate` investigator for codebase exploration
- Feeds into `ws:review` after implementation
