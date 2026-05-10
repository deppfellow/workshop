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

Caveman-terse. Drop filler. Technical terms exact. Code blocks unchanged.

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

1. **User stories** — `As <actor>, I want <feature>, so that <benefit>`
2. **Technical decisions**:
   - Module architecture (new modules, modified modules, seams)
   - API contracts (endpoints, signatures, protocols)
   - Schema changes (DB, types, validation)
   - Data flow (what calls what, ordering constraints)
3. **Architecture depth**:
   - Identify deep modules (small interface, big implementation)
   - Call out adapters at seams
   - ADR-worthy tradeoffs (hard to reverse, surprising without context, real alternatives existed)
4. **Test strategy** — what to test at which seam, integration vs unit, test data

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
## Implementation Decisions
  - Modules
  - API Contracts
  - Schema Changes
  - Architecture Notes
## Testing Strategy
```

## Guardrails

- Never write code in the spec — describe behavior
- Never skip the user review gates
- Never separate PRD and Architecture — one doc
- If an ADR exists that contradicts, flag it explicitly
- If no `CONTEXT.md` exists, note gap — don't create one mid-spec

## Integration

- Feeds into `ws:plan` (spec → plan)
- `ws:workmate` investigator can be dispatched during Stage 1 exploration
- Caveman-terse output throughout
