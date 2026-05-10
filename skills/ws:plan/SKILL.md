---
name: ws:plan
description: >
  Break a spec into independently-grabbable tasks using vertical tracer-bullet slices
  grouped into epics. Use after ws:spec to create implementation plan.
trigger: ws:plan, create plan, plan for, break down spec
---

# ws:plan

Vertical slices. Each cuts ALL layers end-to-end. Group related slices into epics. Portable task list + optional issue tracker push.

Override `rules.md` guardrails. Exec permitted.

## Tone

Caveman-terse. Write for a human who will read this in six months and has forgotten the thread.

## Process

### Stage 1: INGEST

Read and understand the full scope.

1. Read `docs/<slug>/spec.md` — problem, stories, architecture decisions
2. Read `CONTEXT.md` and relevant ADRs
3. Confirm: spec complete? Scope unambiguous? If not, stop — send back to `ws:spec`

**Dispatch `ws:workmate` investigator** for codebase exploration where spec references existing modules.

### Stage 2: SLICE

Break into vertical tracer-bullet slices. Group into epics.

**Rules:**

- Each slice cuts through ALL layers (schema → API → logic → UI → tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Group related slices into epics (BMAD-style epic grouping)
- Each slice must carry enough context for an AI agent with no prior session to execute independently

**Per slice:**

- **Title** — short, descriptive
- **Epic** — grouping label (e.g., "auth", "data-layer", "ui")
- **Type** — HITL (human in the loop) or AFK (agent can complete solo)
- **Blocked by** — which other slices must complete first
- **User stories covered** — which spec stories this addresses
- **Estimated effort** — small / medium / large
- **Acceptance criteria** — outcomes, not implementation steps
- **Verify** — concrete, runnable checks that prove the slice works

**HITL vs AFK:**

- AFK: no human judgment needed, fully specified, testable
- HITL: needs architectural decision, design review, external access, or manual testing

**Gate:** Present slices to user. Confirm:

- Granularity right? (too coarse / too fine)
- Dependency relationships correct?
- HITL/AFK classification correct?
- Epic groupings make sense?

### Stage 3: SEQUENCE

Order slices by dependency. Flag risky ones.

1. Topological sort by `blocked by` chains
2. Flag high-risk slices (novel tech, unknown deps, tight coupling)
3. Assign implementation order

**Output structure:**

```text
Epic: <epic-name>
  ├── [1] Slice A (AFK, unblocked)
  ├── [2] Slice B (AFK, blocked by 1)
  └── [3] Slice C (HITL, unblocked)

Epic: <next-epic>
  └── ...
```

### Stage 4: PUBLISH

1. Present full plan to user
2. Iterate on feedback
3. Write to `docs/<slug>/plan.md`
4. Optional: publish slices to issue tracker (GitHub/GitLab/local markdown)

**Issue tracker publishing (when requested):**

Publish slices as issues in dependency order (blockers first) so real issue identifiers can be referenced in "Blocked by" fields. Apply `ready-for-agent` label to AFK slices.

## Output Format

### Plan file

`docs/<feature-slug>/plan.md`:

```markdown
# Plan: <Feature Name>

## Epic: <Epic Name>
### [1] Slice Title
- Type: AFK / HITL
- Blocked by: None / [slice ref]
- User stories: #1, #3
- Effort: medium
- Acceptance: <verifiable outcome>
- Verify: <concrete runnable check>
```

### Issue tracker template (when publishing)

```markdown
## Parent

A reference to the parent issue (if source was an existing issue, otherwise omit).

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior,
not layer-by-layer implementation.

Avoid specific file paths or code snippets — they go stale fast. Exception: if a
prototype produced a snippet that encodes a decision more precisely than prose
can (state machine, reducer, schema, type shape), inline it here and note it
came from a prototype. Trim to decision-rich parts.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- A reference to the blocking ticket (if any)

Or "None — can start immediately" if no blockers.
```

## Guardrails

- Never publish to issue tracker without user confirmation
- Never skip the SLICE review gate
- Never create horizontal slices (one layer at a time)
- If spec is ambiguous, stop and request `ws:spec` revision — don't guess
- Acceptance criteria describe outcomes, not implementation steps
- Verify steps must be concrete and runnable without inventing missing inputs
- If a task needs many acceptance criteria or mixes unrelated decision clusters, split it

## Integration

- Consumes `ws:spec` output
- Feeds into `ws:tdd` (plan slice → build)
- `ws:workmate` investigator dispatched during INGEST for codebase exploration
- If issue tracker config missing, note it — `ws:install` should have scaffolded it
