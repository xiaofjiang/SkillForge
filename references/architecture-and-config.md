# Architecture Pattern Selection & Configuration

Moved out of SKILL.md during the 2026-07 trim (AUDIT-028) — supplementary reference material for the generation phase, not needed on every invocation.

## Architecture Pattern Selection

Select based on task complexity:

| Pattern | Use When | Structure |
|---------|----------|-----------|
| **Single-Phase** | Simple linear tasks | Steps 1-2-3 |
| **Checklist** | Quality/compliance audits | ☐ Item verification |
| **Generator** | Creating artifacts | Input → Transform → Output |
| **Multi-Phase** | Complex ordered workflows | Phase 1 → Phase 2 → Phase 3 |
| **Multi-Agent Parallel** | Independent subtasks | Launch agents concurrently |
| **Multi-Agent Sequential** | Dependent subtasks | Agent 1 → Agent 2 → Agent 3 |
| **Orchestrator** | Coordinating multiple skills | Meta-skill chains |

### Selection Decision Tree

```
Is it a simple procedure?
├── Yes → Single-Phase
└── No → Does it produce artifacts?
    ├── Yes → Generator
    └── No → Does it verify/audit?
        ├── Yes → Checklist
        └── No → Are subtasks independent?
            ├── Yes → Multi-Agent Parallel
            └── No → Multi-Agent Sequential or Multi-Phase
```

## Configuration

Aspirational config reference (not read by any script in this repo as of the 2026-07 trim — grepped, zero hits — but kept as the documented intent for `mode`/`depth`/panel-size defaults if a future script consumes it):

```yaml
SKILLCREATOR_CONFIG:
  mode: autonomous
  depth: maximum  # always

  analysis:
    min_lens_depth: 5
    max_questioning_rounds: 7
    termination_empty_rounds: 3

  synthesis:
    panel_size: 3
    require_unanimous: true
    max_iterations: 5
    escalate_to_human: true

  evolution:
    min_timelessness_score: 7
    min_extension_points: 2
    require_temporal_projection: true

  model:
    primary: claude-opus-4-5-20251101
    subagents: claude-opus-4-5-20251101
```
