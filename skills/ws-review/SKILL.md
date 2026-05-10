---
name: ws-review
description: >
  Review code and specs through 4 lenses: correctness, security, simplicity, test reality.
  Terse one-liner findings. Verdict: APPROVED or CHANGES REQUESTED.
  Use after ws-tdd, before merge, or on any diff/PR.
trigger: ws-review, review this, code review, review diff
---

# ws-review

4 lenses. One-line findings. No throat-clearing. Tests are real or they're theatre.

## Tone

Caveman-terse. Format: `L42: 🔴 bug: user null. Add guard.`

## Process

### Stage 1: SCAN

Understand what changed and why.

1. Read diff/PR — what files changed, what was added/removed
2. Read the plan slice this implements (from `docs/<slug>/plan.md`)
3. Read the spec for acceptance criteria (from `docs/<slug>/spec.md`)
4. Read `REVIEW.md` from repo root if it exists — apply project-specific concerns
5. If reviewing a PR, read open review comments and classify each: real issue, style preference, or false positive
6. Check: does the diff match the plan slice? Stray changes?

**Dispatch `ws-workmate` investigator** for unfamiliar code paths.

### Stage 2: CHECK

4 lenses. Each pass is a separate review pass.

**Lens 1 — Correctness:**

- Does code do what spec says?
- Edge cases handled? (null, empty, boundary, concurrent)
- Error paths covered?
- Data flow: correct at every step?

**Lens 2 — Security:**

- Injection vectors? (SQL, XSS, command, path traversal)
- Auth/authz: every endpoint guarded?
- Secrets: hardcoded? logged? in version control?
- Input validation: trust nothing from outside

**Lens 3 — Simplicity:**

- Could this be simpler? (deletion test: delete code → does complexity vanish or reappear?)
- Over-engineered? (abstractions for hypothetical futures)
- Duplication? (same pattern in 3 places = extract)
- Deep or shallow? (interface nearly as complex as implementation = shallow)

**Lens 4 — Test Reality:**

- Tests prove acceptance criteria? (not just line coverage)
- Tests use public interfaces? (not internal implementation)
- Tests would survive internal refactor?
- Missing: happy path? sad path? edge case?
- Theatre tests? (asserts nothing meaningful, or tests the mock)

**Dispatch `ws-workmate` reviewer** for per-file findings.

### Stage 3: REPORT

Terse findings. Verdict.

**Format:**

```markdown
## ws-review

### 🔴 Must Fix
L42: bug — user null on empty session. Add guard before L42.
L108: security — raw SQL concat. Use parameterized query.
L15: test theatre — assert(true) proves nothing. Assert cart total.

### 🟡 Should Fix
L67: duplication — same date parse in 3 places. Extract to shared util.
L200: shallow — ConfigLoader interface 12 params, impl 14 lines. Deepen.

### 🟢 Nice
Module depth good. Test seams clear.

### Verdict
CHANGES REQUESTED — 2 🔴, 2 🟡.
```

**Verdicts:**

- **APPROVED** — all lenses pass, no 🔴 findings
- **CHANGES REQUESTED** — 🔴 or significant 🟡 findings

## Guardrails

- Never review code without understanding the spec/plan it implements
- Never approve with 🔴 findings (unless user explicitly overrides)
- Never write implementation in review — findings only
- Reviewer subagent: one-line per finding, no prose
- If spec is missing or outdated, flag it — don't review against stale instructions
- Flag decisions made in code that should have been surfaced in the request, spec, plan, or review context
- Flag broken invariants and silent contract changes from the available context
- Be direct about uncertainty; do not speculate without evidence
- Tie every finding to concrete evidence

## Integration

- Consumes output from `ws-tdd` or any PR/diff
- Dispatches `ws-workmate` investigator (unfamiliar code) and reviewer (per-file findings)
- Feeds into merge decision
- Findings format compatible with `ws-diagnose` if post-merge bug found
