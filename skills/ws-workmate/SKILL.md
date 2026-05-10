---
name: ws-workmate
description: >
  Token-efficient specialized subagents dispatched by other ws-* skills.
  Three roles: investigator (read-only locator), builder (surgical 1-2 file edits),
  reviewer (one-line findings). ~60% fewer tokens than vanilla agents.
  Not invoked directly — dispatched by ws-spec, ws-plan, ws-tdd, ws-review, ws-diagnose.
trigger: ws-workmate, dispatch workmate, spawn workmate
---

# ws-workmate

Specialized subagents. Token-efficient. Constrained scope. Each does ONE thing well.

## Why this exists

Subagent tool results get injected into main context verbatim. A vanilla Explore that returns 2k tokens of prose costs 2k tokens of main-context budget every time. The same finding from a workmate investigator returns ~700 tokens. Across 20 delegations in one session that's the difference between context exhaustion and finishing the task.

**Rule of thumb:** if you'd want the subagent's output in 1/3 the tokens, pick workmate. If you'd want prose, pick vanilla.

## Tone

Caveman-terse. Output ~60% fewer tokens than vanilla agent.

## When to use workmate vs alternatives

| Task | Use |
| --- | --- |
| "Where is X defined / what calls Y / list uses of Z" | workmate investigator |
| Same but you also want suggestions/architecture commentary | Explore (vanilla) |
| Surgical edit, ≤2 files, scope obvious | workmate builder |
| New feature / 3+ files / cross-cutting refactor | Main agent |
| Review diff, branch, or file for bugs | workmate reviewer |
| Deep code review with rationale + alternatives | Main agent |
| One-line answer you already know | Main thread, no subagent |

**Wrong dispatch = wasted tokens.** If task doesn't fit a role, main agent does it.

## Roles

### Investigator

- **Role:** Read-only code locator
- **Answers:** "where is X?" "trace this path" "what calls this?" "find all references to Y"
- **Output:** File paths, line numbers, call chains, data flow. No edits.
- **Constraint:** NEVER edits files. Read-only.
- **Dispatched by:** `ws-diagnose` (LOOP stage), `ws-plan` (INGEST stage), `ws-review` (SCAN stage), `ws-spec` (DEFINE stage)

### Builder

- **Role:** Surgical 1-2 file edits
- **Answers:** "fix this one function" "add this single method" "rename this symbol in 2 files"
- **Output:** Edited files. Tests pass.
- **Constraint:** Refuses >2 files. If change needs 3+ files — reject, escalate to main agent.
- **Dispatched by:** `ws-tdd` (BUILD stage), `ws-diagnose` (INSTRUMENT stage)

### Reviewer

- **Role:** One-line findings per issue
- **Answers:** "review this diff" "check this file for security" "are these tests real?"
- **Output:** Terse findings. Format: `L42: 🔴 bug: user null. Add guard.` No prose.
- **Constraint:** Never implements fixes. Findings only.
- **Dispatched by:** `ws-review` (CHECK stage)

## Output contracts

What main thread can rely on per role:

**Investigator:**
```
<Header>:
- path:line — `symbol` — short note
totals: <counts>.
```
Or `No match.` Always file-path-first, line-number-attached, backticked symbols. Safe to grep with `path:\d+`.

**Builder:**
```
<path:line-range> — <change ≤10 words>.
verified: <re-read OK | mismatch @ path:line>.
```
Or one of: `too-big.` / `needs-confirm.` / `ambiguous.` / `regressed.` (terminal first token).

**Reviewer:**
```
path:line: <emoji> <severity>: <problem>. <fix>.
totals: N🔴 N🟡 N🔵 N❓
```
Or `No issues.` Findings sorted file → line ascending.

## Process

### Stage 1: ASSIGN

Match task to correct role.

| Task pattern | Role |
| --- | --- |
| Find, trace, locate, search, map | investigator |
| Fix, add, modify, rename (≤2 files) | builder |
| Review, check, audit, verify | reviewer |
| Multi-file change, architecture decision, complex refactor | DO NOT dispatch — main agent handles |

### Stage 2: BRIEF

Write tight, scoped prompt for the subagent.

**Rules:**

- Exact task, no ambiguity
- Specific files/paths when known
- Acceptance criteria (what "done" looks like)
- Minimal context — only what the subagent needs
- Never dump full codebase into subagent context

**Example briefs:**

```text
Investigator: "Find where user session token is validated in src/auth/. Trace the
call chain from middleware to token verification. Report file:line for each step."

Builder: "In src/auth/middleware.ts:42, fix token expiry check from `<` to `<=`.
Also update src/auth/middleware.test.ts:67 to test the boundary case. Tests must pass."

Reviewer: "Review diff in src/payment/. Check for: injection vectors, missing null
guards, test assertions that don't actually assert. One finding per line."
```

### Stage 3: DISPATCH

Launch subagent. Consume output back into main context.

- Investigator output → main agent uses for decision-making
- Builder output → main agent verifies (tests pass, scope respected)
- Reviewer output → main agent synthesizes into `ws-review` report

**Post-dispatch:** verify subagent stayed in scope. Escalate if not.

## Chaining patterns

**Locate → fix → verify** (most common):
1. workmate investigator returns site list.
2. Main thread picks 1-2 sites, hands paths to workmate builder.
3. workmate reviewer audits the diff.

**Parallel scout** (when investigation is broad):
Spawn 2-3 workmate investigator calls in one message (different angles: defs vs callers vs tests). Aggregate in main thread.

**Single-shot edit** (when site is already known):
Skip investigator. Hand exact path:line to workmate builder directly.

## Subagent Constraints

All roles:

- Caveman-terse output (~60% fewer tokens)
- Never exceed scope
- Never make decisions reserved for main agent
- Auto-clarity: drop caveman → normal English for security warnings, irreversible-action confirmations, and any output where fragment ambiguity could be misread. Resume caveman after.

Builder specific:

- 1-2 files max. Refuse 3+.
- Must include test changes
- Must verify tests pass before reporting done

Investigator specific:

- Read-only. Any attempted edit = violation.
- Report exact file:line references

Reviewer specific:

- Findings only. No implementation.
- One line per finding. No grouping, no summaries.

## What NOT to do

- Don't use workmate builder when you don't already know the file. Spawn investigator first or main thread will eat tokens passing context.
- Don't chain investigator → builder for a 5-file refactor. Builder will return `too-big.` and you'll have wasted a turn.
- Don't ask reviewer for "general feedback" — it returns findings only, no architecture opinions.
- Don't expect prose. Workmate output is structured, sometimes terse to the point of cryptic. If a human will read it directly, paraphrase.

## Integration

Not invoked directly by user — dispatched by other ws-* skills:

- `ws-spec` → investigator (DEFINE stage)
- `ws-plan` → investigator (INGEST stage)
- `ws-tdd` → investigator (SETUP) + builder (BUILD)
- `ws-review` → investigator (SCAN) + reviewer (CHECK)
- `ws-diagnose` → investigator (LOOP) + builder (INSTRUMENT)
