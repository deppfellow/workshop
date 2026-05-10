---
name: ws-install
description: >
  Scaffold the workshop repo structure and install ws-* hybrid skills.
  Detects IDE/agent, creates directory layout, writes skill files.
  No external provider dependencies — ws-* skills are standalone.
trigger: ws-install, setup workshop, install ws, scaffold workshop
---

# ws-install

Scaffold full repo. Install ws-* skills. Write config. Detect agent.

No external provider install. ws-* skills are standalone.

## Tone

Caveman-terse.

## Process

### Stage 1: DETECT

Identify environment and existing state.

**Agent detection:**

- Claude Code: `.claude/` exists, `claude` in PATH
- Cursor: `.cursor/` exists
- Windsurf: `.windsurf/` exists
- Codex: `.codex/` exists, `codex` in PATH
- GitHub Copilot: `.github/copilot-instructions.md` exists
- opencode: currently active tool
- Generic: `AGENTS.md` or `CLAUDE.md` at root

**Existing state:**

- `.docs/` — workshop skills already installed?
- `.agents/skills/` — other skills present?
- `skills-lock.json` — provider tracking?
- `AGENTS.md` / `CLAUDE.md` — agent config exists?
- `CONTEXT.md` — domain glossary exists?
- `docs/adr/` — architecture decisions?

Report findings. Confirm with user before writing.

### Stage 2: SCAFFOLD

Create repo structure. Write config files.

**Directory layout:**

```text
project-root/
├── .docs/
│   ├── ws-brainstorm/SKILL.md
│   ├── ws-brainstorm-ref-docs/SKILL.md
│   ├── ws-direct-talk/SKILL.md
│   ├── ws-spec/SKILL.md
│   ├── ws-plan/SKILL.md
│   ├── ws-tdd/SKILL.md
│   ├── ws-review/SKILL.md
│   ├── ws-diagnose/SKILL.md
│   ├── ws-improvement/SKILL.md
│   ├── ws-workmate/SKILL.md
│   └── ws-install/SKILL.md
├── docs/
│   ├── adr/                    # Architecture Decision Records
│   └── agents/                 # Agent skill references
│       ├── issue-tracker.md
│       ├── triage-labels.md
│       └── domain.md
├── CONTEXT.md                  # Domain glossary (created if absent)
├── AGENTS.md or CLAUDE.md     # Agent config with workshop block
└── skills-lock.json            # Workshop skill manifest
```

**Files to write:**

`CONTEXT.md` (if absent):

```markdown
# Domain Glossary

<!-- Define project-specific terms here. -->
<!-- Used by ws-spec, ws-plan, ws-diagnose for consistent language. -->
```

`AGENTS.md` or `CLAUDE.md` (append workshop block):

```markdown
## Workshop Skills (ws-*)

Installed in `.docs/`. Hybrid engineering skills blending SDLC workflow,
disciplined engineering, and terse communication.

| Skill | Purpose | Invoke |
|---|---|---|
| ws-brainstorm | Relentless interview (stress-test plans) | ws-brainstorm |
| ws-brainstorm-ref-docs | Docs-aware interview (updates CONTEXT.md + ADRs) | ws-brainstorm-ref-docs |
| ws-direct-talk | Compact communication mode (lite/full) | ws-direct-talk |
| ws-spec | Spec writer (requirements + design) | ws-spec |
| ws-plan | Task breakdown (vertical slices + epics) | ws-plan |
| ws-tdd | Test-driven development | ws-tdd |
| ws-review | Code review (4 lenses) | ws-review |
| ws-diagnose | Bug diagnosis (5-stage loop) | ws-diagnose |
| ws-improvement | Architecture deepening | ws-improvement |
| ws-workmate | Subagent orchestration | ws-workmate |
| ws-install | Workshop setup | ws-install |

See `docs/agents/` for issue tracker, triage labels, and domain doc layout.
```

`docs/agents/domain.md`:

```markdown
# Domain Docs

Single-context layout:
- CONTEXT.md at project root — domain glossary
- docs/adr/ — architecture decision records

All ws-* skills read these for vocabulary and past decisions.
```

`skills-lock.json`:

```json
{
  "version": 1,
  "skills": {
    "ws-brainstorm": { "source": "local", "skillPath": ".docs/ws-brainstorm/SKILL.md" },
    "ws-brainstorm-ref-docs": { "source": "local", "skillPath": ".docs/ws-brainstorm-ref-docs/SKILL.md" },
    "ws-direct-talk": { "source": "local", "skillPath": ".docs/ws-direct-talk/SKILL.md" },
    "ws-spec": { "source": "local", "skillPath": ".docs/ws-spec/SKILL.md" },
    "ws-plan": { "source": "local", "skillPath": ".docs/ws-plan/SKILL.md" },
    "ws-tdd": { "source": "local", "skillPath": ".docs/ws-tdd/SKILL.md" },
    "ws-review": { "source": "local", "skillPath": ".docs/ws-review/SKILL.md" },
    "ws-diagnose": { "source": "local", "skillPath": ".docs/ws-diagnose/SKILL.md" },
    "ws-improvement": { "source": "local", "skillPath": ".docs/ws-improvement/SKILL.md" },
    "ws-workmate": { "source": "local", "skillPath": ".docs/ws-workmate/SKILL.md" },
    "ws-install": { "source": "local", "skillPath": ".docs/ws-install/SKILL.md" }
  }
}
```

### Stage 3: VERIFY

Post-install checks.

- [ ] All 11 ws-* SKILL.md files present in `.docs/`
- [ ] `skills-lock.json` valid and matches installed skills
- [ ] `AGENTS.md` or `CLAUDE.md` has workshop block
- [ ] `CONTEXT.md` exists
- [ ] `docs/adr/` and `docs/agents/` directories exist
- [ ] No conflicts with existing `.agents/skills/`

**Report:**

- Agent detected: <agent>
- Skills installed: 11/11
- Next step: `ws-spec` to start feature work

## Guardrails

- Never overwrite existing `CONTEXT.md` — append if needed
- Never delete existing `AGENTS.md` / `CLAUDE.md` — append workshop block
- Never install external providers (blueprint, mattpocock, caveman) — ws-* only
- Never write to `docs/adr/` — create directory, leave empty for user

## Integration

- First skill to run in any workshop-enabled project
- Creates directory structure consumed by all other ws-* skills
- `skills-lock.json` used by ws-install to check installation state on re-run
- Detects but does not modify `.agents/skills/` — coexists with existing skills
