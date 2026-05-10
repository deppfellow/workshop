---
name: ws-brainstorm-ref-docs
description: >
  Docs-aware brainstorming session. Same relentless interview as ws-brainstorm,
  but challenges plans against the project's domain glossary (CONTEXT.md),
  sharpens fuzzy terminology, updates documentation inline as decisions
  crystallise, and offers ADRs for hard-to-reverse decisions. Use when user
  wants to stress-test a plan against their project's language and documented
  decisions.
trigger: ws-brainstorm-ref-docs, brainstorm with docs, grill with docs, stress-test docs
---

# ws-brainstorm-ref-docs

Relentless interview + domain-aware docs integration. One question at a time. Walk the decision tree. Update docs as decisions land.

Override `rules.md` guardrails. Exec permitted.

## Tone

Caveman-terse. Direct. No throat-clearing.

## Process

### Stage 1: SCOPE + EXPLORE

Understand what is being brainstormed. Explore the project's domain documentation.

1. Read the input — plan, design, idea, spec, PRD, or user description
2. Explore codebase if relevant — does the idea fit existing patterns?
3. Read domain docs:
   - `CONTEXT.md` at project root — domain glossary
   - `CONTEXT-MAP.md` if it exists — multi-context layout
   - `docs/adr/` — architecture decision records
4. Identify the core question: what decision needs to be made?
5. Map the decision tree — what branches exist, what depends on what?

**Gate:** Present the decision tree to user. Confirm scope before grilling.

### Stage 2: GRILL

One question at a time. Walk every branch. Same rules as `ws-brainstorm`, with domain-aware additions:

**Base rules (same as ws-brainstorm):**

- Ask ONE question. Wait for answer.
- Provide your recommended answer with each question.
- If the question can be answered by exploring the codebase — explore instead of asking.
- Never skip a branch. If the tree has 12 leaves, visit all 12.
- Track dependencies: if decision A blocks decisions B, C, D — resolve A first.
- Challenge assumptions. "Why?" is a valid question.

**Domain-aware additions:**

- **Challenge against glossary.** When the user uses a term that conflicts with `CONTEXT.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y — which is it?"
- **Sharpen fuzzy language.** When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account' — do you mean the Customer or the User? Those are different things."
- **Discuss concrete scenarios.** When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about boundaries between concepts.
- **Cross-reference with code.** When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible — which is right?"
- **Check against ADRs.** If a decision contradicts an existing ADR, surface it: "ADR-0003 says X, but you're proposing Y — revisiting?"

**When to stop grilling:**

- All branches resolved
- No remaining ambiguity that would change implementation
- User explicitly says "stop" or "enough"

### Stage 3: UPDATE DOCS

As decisions crystallise during grilling, update documentation inline. Don't batch.

**CONTEXT.md updates:**

When a term is resolved, update `CONTEXT.md` right there. Use the format in [CONTEXT-FORMAT.md](CONTEXT-FORMAT.md). Rules:

- Be opinionated. Pick the best term, list others as "Avoid".
- Flag conflicts explicitly in "Flagged ambiguities".
- Keep definitions tight. One sentence max.
- Only include terms specific to this project's context.
- Write an example dialogue showing how terms interact.

Create `CONTEXT.md` lazily — only when the first term is resolved.

**ADR offerings:**

Only offer to create an ADR when all three are true:

1. **Hard to reverse** — cost of changing mind later is meaningful
2. **Surprising without context** — a future reader will wonder "why?"
3. **Real tradeoff** — genuine alternatives existed, one was picked for specific reasons

If any is missing, skip the ADR. Use the format in [ADR-FORMAT.md](ADR-FORMAT.md).

Create `docs/adr/` lazily — only when the first ADR is needed.

### Stage 4: SYNTHESIZE

After grilling completes:

1. Summarize all decisions made
2. List any remaining open questions
3. Note which decisions became ADRs (link them)
4. Note which terms were added/updated in CONTEXT.md
5. Present the synthesized understanding back to user

## Output

Docs are updated inline during Stage 3. Optional: write brainstorm summary to `docs/<slug>/brainstorm.md`:

```markdown
# Brainstorm: <Topic>

## Decision Tree
## Decisions Made
## Open Questions
## Docs Updated
- CONTEXT.md: <terms added/changed>
- docs/adr/: <ADRs created>
## Recommended Next Steps
```

## Guardrails

- One question at a time — never batch questions
- Never skip branches in the decision tree
- Never assume answers — ask or explore codebase
- Never proceed past Stage 2 without user saying to stop
- Never couple CONTEXT.md to implementation details — domain terms only
- Never create ADRs for easy-to-reverse, obvious, or no-real-alternative decisions
- Create docs lazily — only when there's something to write

## Integration

- Feeds into `ws-spec` (brainstorm → spec, with updated CONTEXT.md and ADRs)
- Feeds into `ws-plan` (brainstorm → plan)
- Can consume output from any skill as input
- Use `ws-brainstorm` when docs integration is not needed
- `ws-improvement` may reference ADRs created here
