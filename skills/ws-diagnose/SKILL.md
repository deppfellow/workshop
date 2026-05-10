---
name: ws-diagnose
description: >
  Disciplined diagnosis loop for hard bugs and performance regressions.
  Reproduce → hypothesise → instrument → fix → regression-test.
  Use when bug reported, something broken/throwing/failing, or performance regression.
trigger: ws-diagnose, diagnose this, debug this, find the bug, what's broken
---

# ws-diagnose

5-stage loop. Feedback signal first — that's THE skill. No loop = no proceed.

## Tone

Caveman-terse. Errors quoted exact.

## Process

### Stage 1: LOOP

Build fast, deterministic pass/fail signal. **This is the skill.** Everything else mechanical.

Spend disproportionate effort here. Be aggressive. Be creative. Refuse to give up.

**Construct loop — try in order:**

1. **Failing test** — at whatever seam reaches the bug (unit, integration, e2e)
2. **cURL / HTTP script** — against running dev server
3. **CLI invocation** — fixture input, diff stdout against known-good snapshot
4. **Headless browser** — Playwright/Puppeteer, assert on DOM/console/network
5. **Replay captured trace** — saved request/payload/event log, replay in isolation
6. **Throwaway harness** — minimal subsystem subset, mocked deps, single fn call
7. **Property / fuzz loop** — 1000 random inputs, find failure mode
8. **Bisection harness** — automate `git bisect run` between known states
9. **Differential loop** — same input, old-version vs new-version, diff outputs
10. **HITL script** — last resort, drive human with structured loop template

**Iterate on the loop:**

- Make it faster (cache setup, skip unrelated init)
- Make signal sharper (assert on specific symptom, not "didn't crash")
- Make it deterministic (pin time, seed RNG, isolate filesystem)

**Non-deterministic bugs:** goal is higher reproduction rate. Loop 100×, parallelise, add stress, narrow timing windows. 50% flake = debuggable. 1% = not.

**If no loop possible:** stop. Say so. List what tried. Ask user for: env access, captured artifact (HAR, log dump, core dump), or permission for temp instrumentation.

**Dispatch `ws-workmate` investigator** for codebase exploration — trace the bug path.

**Gate:** loop exists and produces failure. Do not proceed without.

### Stage 2: REPRO

Run the loop. Watch bug appear.

Confirm:

- Failure mode matches USER's description — not a nearby different failure
- Reproducible across multiple runs (or high enough rate for flaky bugs)
- Exact symptom captured (error message, wrong output, slow timing)

**Gate:** bug reproduced. Symptom matches user report.

### Stage 3: HYPOTHESISE

Generate **3-5 ranked hypotheses** before testing any. Single-hypothesis generation anchors on first plausible idea.

Each hypothesis must be **falsifiable**:

> "If `<X>` is the cause, then `<changing Y>` will make the bug disappear / `<changing Z>` will make it worse."

If you can't state the prediction — discard or sharpen it. Not a vibe.

**Show ranked list to user.** Cheap checkpoint. They often have domain knowledge that re-ranks instantly.

### Stage 4: INSTRUMENT

Probe one hypothesis at a time. **Change one variable per attempt.**

Tool preference:

1. **Debugger / REPL** — one breakpoint beats ten logs
2. **Targeted logs** — at boundaries that distinguish hypotheses
3. Never "log everything and grep"

**Tag every debug log:** `[DEBUG-xxxx]`. Cleanup = single grep later. Tagged logs die; untagged survive.

**Perf branch:** For performance regressions, logs usually wrong. Instead: establish baseline (timing harness, `performance.now()`, profiler, query plan), then bisect. Measure first, fix second.

**Dispatch `ws-workmate` builder** for small instrumentation changes.

### Stage 5: FIX

1. **Regression test first** — but only if correct seam exists
2. A correct seam: test exercises REAL bug pattern at call site. Shallow seam (single-caller test when bug needs multiple callers) = false confidence
3. **If no correct seam exists** — that IS the finding. Note it. Architecture prevents bug from being locked down. Flag for `ws-spec` revision.
4. If correct seam exists:
   - Turn minimised repro into failing test at that seam
   - Watch it fail
   - Apply fix
   - Watch it pass
   - Re-run Stage 1 loop against original (un-minimised) scenario

**Cleanup — required before done:**

- [ ] Original repro no longer reproduces (re-run Stage 1 loop)
- [ ] Regression test passes (or absence of seam documented)
- [ ] All `[DEBUG-xxxx]` instrumentation removed
- [ ] Throwaway prototypes deleted (or moved to clearly-marked debug location)
- [ ] Hypothesis that was correct stated in commit/PR message

**Post-mortem:** what would have prevented this bug? If the answer involves architectural change (no good test seam, tangled callers, hidden coupling) — hand off to `ws-improvement` with the specifics. Make the recommendation **after** the fix is in, not before — you have more information now than when you started.

## Guardrails

- Never proceed past Stage 1 without a working loop
- Never proceed past Stage 2 without reproducing the actual bug
- Never test hypotheses before ranking them
- Never skip debug log cleanup
- Stage 5 FIX: regression test at correct seam or document its absence

## Integration

- May consume output from `ws-spec`, `ws-plan`, or any bug report
- Dispatches `ws-workmate` investigator (codebase tracing) and builder (instrumentation)
- Feeds architectural findings to `ws-improvement` for deepening or `ws-spec` for revision
- Cleanup step produces commit message including correct hypothesis
