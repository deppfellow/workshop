---
name: ws:spec
description: >
  Write a spec — requirements and technical design in one document.
  Use when starting new feature work or when design decisions matter.
  Invoke with "ws:spec", "write a spec", or "create spec for...".
trigger: ws:spec, write a spec, create spec, spec for
---

# ws:spec

One document. Requirements AND design together. Spec is prompt with weight — once code right, spec job done.

Override `rules.md` guardrails. Exec permitted.

## Tone

Caveman-terse. Drop filler. Technical terms exact. Code blocks unchanged. Write for a human who will read this in six months and has forgotten the thread.

## Process

### Stage 1: DEFINE

Explore then define. Don't design yet.

1. **Scan codebase** — `CONTEXT.md`, `docs/adr/`, existing code in area
2. **Read input** — any provided brief, PRD, or user description
3. **Write problem statement** — user's perspective, what's broken/missing
4. **Set success criteria** — measurable, verifiable
5. **Declare out of scope** — what this spec does NOT cover

**Gate:** Present problem + scope to user. Confirm before designing.

### Stage 2: DESIGN

Deep architecture. Concrete decisions.

1. **User stories** — numbered, extensive, covering all aspects:
   - Format: `As <actor>, I want <feature>, so that <benefit>`
   - Cover every aspect of the feature — happy path, edge cases, error paths
   - Be exhaustive — this list drives implementation completeness
2. **Module architecture**:
   - New modules, modified modules, seams
   - **Actively look for deep module opportunities** — small interface, big implementation
   - Identify what callers get from depth (leverage) and what maintainers get (locality)
   - Call out adapters at seams
3. **Technical decisions**:
   - API contracts (endpoints, signatures, protocols)
   - Schema changes (DB, types, validation)
   - Data flow (what calls what, ordering constraints)
4. **Invariants** *(when relevant)* — what must not break, and how to check it
5. **Error behavior** *(when relevant)* — failure paths, error shapes, recovery
6. **ADR-worthy tradeoffs** — hard to reverse, surprising without context, real alternatives existed

**Gate:** Present design to user. Iterate until approved.

### Stage 3: WRITE

1. Draft spec from Stages 1+2
2. Present full spec to user
3. Iterate on feedback
4. Write to `docs/<slug>/spec.md`

## Output Format

`docs/<feature-slug>/spec.md`:

```markdown
# <Feature Name>

## Problem Statement
## Success Criteria
## Out of Scope
## User Stories

Numbered, extensive list covering all aspects of the feature.

## Implementation Decisions
  - Modules (with deep module opportunities noted)
  - API Contracts
  - Schema Changes
  - Architecture Notes

## Invariants
What must not break, and how to check it.

## Error Behavior
Failure paths, error shapes, recovery.

## Testing Strategy
What to test at which seam, integration vs unit, test data.
Prior art — similar tests in the codebase that inform approach.
```

## Rules

- Never write code in the spec — describe behavior
- Never skip the user review gates
- Never separate PRD and Architecture — one doc
- If an ADR exists that contradicts, flag it explicitly
- If no `CONTEXT.md` exists, note gap — don't create one mid-spec
- If a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it in the relevant section and note it came from a prototype — trim to decision-rich parts, not a working demo
- Smallest safe change that fully solves the problem
- If two implementations would behave differently, specify the default
- Don't write a greenfield design if the codebase has patterns to follow

## Integration

- Feeds into `ws:plan` (spec → plan)
- `ws:workmate` investigator can be dispatched during Stage 1 exploration
- `ws:brainstorm` or `ws:brainstorm-ref-docs` can feed into spec
- Caveman-terse output throughout
