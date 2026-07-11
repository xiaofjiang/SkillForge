---
name: skillforge
description: "Intelligent skill router and CREATOR. Analyzes ANY input to recommend an existing skill, route a contained existing-skill edit to skill-edit, or create a new skill (and handle splits/merges). Uses deep iterative analysis with 11 thinking models, regression questioning, evolution lens, and multi-agent synthesis panel. Phase 0 triage ensures you never duplicate existing functionality. NOT for contained edits to an existing skill (use skill-edit)."
license: MIT
metadata:
  version: 4.0.0
  model: claude-opus-4-5-20251101
  subagent_model: claude-opus-4-5-20251101
  domains: [meta-skill, automation, skill-creation, orchestration, agentic, routing]
  type: orchestrator
  inputs: [any-input, user-goal, domain-hints]
  outputs: [SKILL.md, references/, scripts/, SKILL_SPEC.md, recommendations]
---

# SkillForge 4.0 - Intelligent Skill Router & Creator

Analyzes ANY input to find, improve, or create the right skill.

Trimmed 2026-07 (System-Audit AUDIT-028) to its used core: Phase-0 triage, the create/improve decision, the spec+generation workflow, and output conventions. The deep analysis ceremony (11 thinking lenses, regression questioning, multi-agent synthesis panel, consensus protocol, evolution/timelessness scoring, architecture pattern selection, config, changelog) moved to `references/` — load a reference when you need the full framework, not on every invocation. This repo is forked to `xiaofjiang/SkillForge` (AUDIT-049) so local edits like this one persist and sync via normal machine setup instead of living only in an untracked clone (`~/.claude/.gitignore` still excludes `skills/skillforge/`; clone the fork, not upstream — see Extension Points).

---

## Quick Start

**Any input works.** SkillForge figures out what you need:

```
SkillForge: create a skill for automated code review   → creates (after duplicate check)
improve the testgen skill to handle React components   → improvement mode for TestGen
do I have a skill for database migrations?              → recommends DBSchema, database-migration
TypeError: Cannot read property 'map' of undefined      → routes to debugging skills
```

## Triggers

**Creation:** `SkillForge: {goal}` · `create skill` · `design skill for {purpose}` · `ultimate skill` · `skillforge --plan-only`
**Routing:** `{any input}` · `do I have a skill for` · `which skill` / `what skill` · `improve {skill-name} skill` · `help me with` / `I need to`

## Process Overview

```
ANY INPUT → Phase 0: Triage (classify, scan ecosystem, decide) →
  USE_EXISTING / IMPROVE_EXISTING → done
  CREATE_NEW → Phase 1 Analysis → Phase 2 Spec → Phase 3 Generate → Phase 4 Synthesis Panel → done
  COMPOSE / CLARIFY → hand off / ask
```

Phase 1 (deep analysis: 11 thinking lenses, regression questioning, automation analysis) and Phase 4 (multi-agent synthesis panel) are full frameworks documented in `references/` — see the References section below. This file covers Phase 0 (below, in full), and the spec+generation workflow (Phase 2/3) that most invocations actually need.

## Commands

| Command | Action |
|---------|--------|
| `SkillForge: {goal}` | Full autonomous execution |
| `SkillForge --plan-only {goal}` | Generate specification only |
| `SkillForge --quick {goal}` | Reduced depth (not recommended) |
| `SkillForge --triage {input}` | Run Phase 0 triage only |
| `SkillForge --improve {skill}` | Enter improvement mode for existing skill |

---

## Step 0: Change-loop on-ramp (do this FIRST, before Phase 0)

Before creating or materially editing a skill, load `~/.claude/references/Change-Loop.md` and state three things up front (the build-new on-ramp from the global CLAUDE.md, mechanized here so it fires by trigger, not memory):

1. **Expectation**, one sentence: "after this skill exists / this edit lands, X should happen."
2. **Reversibility plan**: dry-run the edit, archive-not-delete, one atomic commit per change.
3. **Verification step**: how you will confirm it worked, which for skills is the pre-deploy subagent test below.

Then proceed to triage.

## Standards every skill must meet (enforce at generation)

These are non-negotiable and apply to every CREATE_NEW and IMPROVE_EXISTING path. Check them in the Phase-4 panel review, not just at the end.

- **Never embed an integration recipe. Point at the Integration Registry.** If the skill touches ANY external service (TickTick, Notion, Monday, BrightData, Gmail, Dropbox, email, the Photo/Video Bank, the vault-handoff buffer), it must NOT copy the call mechanics (token load, curl/API syntax, MCP-vs-script choice, endpoint, version headers) into the skill. It says "do X per the Integration-Registry [Service] section" and keeps only its own workflow logic (which board/DB/column, when, formatting). The mechanics live in exactly one place: `~/.claude/references/Integration-Registry.md`. If the service has no registry section yet, add the section there first, then point at it. This is the registry-first rule from System-Audit I-10; embedding a recipe is the drift bug the registry exists to kill (one MCP rename used to rot ~12 skills silently).

## Phase 0: Skill Triage

Before creating anything, SkillForge intelligently analyzes your input to determine the best action. This is the used core of the skill — nearly every invocation ends here.

### Decision Matrix

| Input classification | Match confidence | Action |
|---|---|---|
| explicit_create, explicit_improve, skill_question, task_request, error_message, code_snippet | Match ≥80% + explicit create | **CLARIFY** (duplicate warning) |
| (any) | Match ≥80% + other input | **USE_EXISTING** (recommend skill) |
| (any) | Match 50-79% | **IMPROVE_EXISTING** (load + enhance) |
| (any) | Match <50% + explicit create | **CREATE_NEW** (proceed to Phase 1) |
| Multi-domain detected | — | **COMPOSE** (suggest skill chain) |
| Ambiguous input | — | **CLARIFY** (ask for more info) |

### Triage Script

```bash
python scripts/triage_skill_request.py "help me debug this error"
python scripts/triage_skill_request.py "create a skill for payments" --json

# → USE_EXISTING: Recommends ErrorExplainer (92%)
# → CLARIFY: CodeReview skill exists (85%), create anyway?
# → COMPOSE: Multi-domain, suggests APIDesign + AuthSystem + TestGen chain
```

### Ecosystem Index

```bash
# Rebuild skill index (run periodically or after installing new skills)
python scripts/discover_skills.py
# Index location: ~/.cache/skillrecommender/skill_index.json
# Scans: ~/.claude/skills/, plugins/marketplaces/*, plugins/cache/*
```

### Integration with Phases 1-4

- **USE_EXISTING**: Exits early, no creation needed
- **IMPROVE_EXISTING**: hand a contained edit to the `skill-edit` skill (its 8-step placement protocol). skillforge stays involved only if the enhancement turns out to need a split, merge, or new sibling skill.
- **CREATE_NEW**: Full pipeline (Phase 1 → 2 → 3 → 4)
- **COMPOSE**: Suggests using SkillComposer instead
- **CLARIFY**: Pauses for user input before proceeding

---

## Spec + Generation Workflow (Phase 2-3)

### Before drafting: gather requirements

Explicit (what was literally asked) + implicit (unstated but expected) + unknown-unknowns (expert-level considerations) + domain context (related/overlapping skills — `ls ~/.claude/skills/`, grep existing triggers). Full 11-lens deep-analysis version of this step: `references/multi-lens-framework.md` and `references/regression-questions.md`.

### Specification structure

Captures the analysis as an XML spec: metadata (name, iterations, timelessness score), context (problem statement, existing landscape), requirements (explicit/implicit/discovered), architecture (pattern + WHY, phases, decision points), scripts (decision + inventory + agentic capabilities), evolution analysis (score, extension points, obsolescence triggers), anti-patterns, success criteria. Full XML template: `references/specification-template.md`.

Validate before Phase 3: all sections present, every decision has a WHY, timelessness score ≥7 with justification, ≥2 extension points, scripts section complete if applicable.

### Generation order

```
1. Create directory structure
   mkdir -p ~/.claude/skills/{skill-name}/references
   mkdir -p ~/.claude/skills/{skill-name}/assets/templates
   mkdir -p ~/.claude/skills/{skill-name}/scripts  # if scripts needed

2. Write SKILL.md: frontmatter, title+intro, Quick Start, Triggers (3-5 phrases),
   Quick Reference table, How It Works, Commands, Scripts section (if applicable),
   Validation section, Anti-Patterns, Verification criteria, Deep Dives in <details>.

3. Generate reference documents (deep documentation, templates, checklists) if needed.

4. Create assets (templates for generated outputs) if needed.

5. Create scripts if needed: Result dataclass pattern, exit codes (0=success,
   1=failure, 10=validation failure, 11=verification failure), self-verification,
   documented in SKILL.md with usage examples. See references/script-integration-framework.md
   for the decision tree (script vs no script) and the 5 agentic patterns
   (self-verification, error recovery, state persistence, structured output,
   graceful degradation).
```

### Quality checks during generation

| Check | Requirement |
|-------|-------------|
| Frontmatter | Only allowed properties (name, description, license, allowed-tools, metadata) |
| Name | Hyphen-case, ≤64 chars |
| Description | ≤1024 chars, no angle brackets |
| Triggers | 3-5 distinct, natural language |
| Phases | 1-3 max, not over-engineered |
| Verification | Concrete, measurable |
| Tables over prose | Structured information in tables |
| No placeholder text | Every section fully written |
| Scripts (if present) | Shebang, docstring, argparse, exit codes, Result pattern |

### Panel review (Phase 4)

3-4 Opus agents (Design/Architecture, Audience/Usability, Evolution/Timelessness, + Script agent if scripts present) review against distinct criteria; unanimous APPROVED required or it returns to Phase 1 with issues (max 5 iterations, then flag for human review). Full panel definitions, scoring rubrics, and the consensus protocol: `references/synthesis-protocol.md`. Timelessness scoring (1-10, ≥7 required) and anti-obsolescence patterns: `references/evolution-scoring.md`. Architecture pattern selection (single-phase / checklist / generator / multi-phase / multi-agent / orchestrator) and the aspirational config block: `references/architecture-and-config.md`.

---

## Validation & Packaging

```bash
python scripts/quick_validate.py ~/.claude/skills/my-skill/     # required for packaging
python scripts/validate-skill.py ~/.claude/skills/my-skill/     # full structural validation
python scripts/package_skill.py ~/.claude/skills/my-skill/ ./dist
```

### Frontmatter Requirements

| Property | Required | Description |
|----------|----------|--------------|
| `name` | Yes | Hyphen-case, max 64 chars |
| `description` | Yes | Max 1024 chars, no angle brackets |
| `license` | No | MIT, Apache-2.0, etc. |
| `allowed-tools` | No | Restrict tool access |
| `metadata` | No | Custom fields (version, model, etc.) |

## Skill Output Structure

```
~/.claude/skills/{skill-name}/
├── SKILL.md                    # Main entry point (required)
├── references/                 # Deep documentation (optional)
├── assets/                     # Templates (optional)
└── scripts/                    # Automation scripts (optional)
    ├── validate.py             # Validation/verification
    ├── generate.py             # Artifact generation
    └── state.py                # State management
```

Scripts enable a skill to be agentic (autonomous, self-verifying). Categories: validation, generation, state management, transformation, calculation. Requirements: Python 3.x stdlib-only with graceful fallbacks, `Result` dataclass pattern, exit codes as above, self-verification where applicable, documented usage in SKILL.md. Full catalog of standard patterns: `references/script-patterns-catalog.md`.

---

## Pre-deploy subagent test (REQUIRED before declaring any skill done)

Any skill this session CREATED or MATERIALLY edited (trigger/description changes, new modes, restructured steps) must be tested with `superpowers:testing-skills-with-subagents` BEFORE it is declared done (audit I-2). This is TDD for skills: a fresh subagent gets realistic trigger phrases and we watch whether the right skill fires and its steps get followed, before the skill goes live. Skills are Xiao's real codebase; the recurring pain classes here (trigger misfires, wrong-pathway defaults surviving an edit) are exactly what a pre-deploy test catches.

- Run it on the just-built/edited skill; feed it the trigger phrases a real user would type plus one or two neighbor-skill phrases that should NOT fire it.
- If the wrong skill fires, or a step is skipped or misread, fix the SKILL.md and re-test until clean. Do not close the work on a failing test.
- Trivial cosmetic edits (a typo, a one-word description tweak) are exempt; anything touching triggers, routing, or steps is not.
- `skill-edit` carries a related step 7, but scoped tighter (a quick subagent test only for boundary/routing edits). This skillforge gate is broader: it applies on the CREATE path and to any material edit (new modes, restructured steps), and names the specific `superpowers:testing-skills-with-subagents` skill rather than the generic pattern.

---

## Anti-Patterns

| Avoid | Why | Instead |
|-------|-----|---------|
| Duplicate skills | Bloats registry | Check existing first |
| Single trigger | Hard to discover | 3-5 varied phrases |
| No verification | Can't confirm success | Measurable outcomes |
| Over-engineering | Complexity without value | Start simple |
| Missing WHY | Can't evolve | Document rationale |
| Invalid frontmatter | Can't package | Use allowed properties only |

## Verification Checklist

- [ ] Frontmatter valid (only allowed properties)
- [ ] Name is hyphen-case, ≤64 chars
- [ ] Description ≤1024 chars, no `<` or `>`
- [ ] 3-5 trigger phrases defined
- [ ] Timelessness score ≥ 7
- [ ] `python scripts/quick_validate.py` passes
- [ ] Created/materially-edited skill passed `superpowers:testing-skills-with-subagents` (right skill fires, steps followed)

---

## References

- [Multi-Lens Framework](references/multi-lens-framework.md) - 11 thinking models, full Phase 1 analysis
- [Regression Questions](references/regression-questions.md) - Complete question bank (7 categories)
- [Specification Template](references/specification-template.md) - Full XML spec structure
- [Script Integration Framework](references/script-integration-framework.md) - When and how to create scripts
- [Script Patterns Catalog](references/script-patterns-catalog.md) - Standard Python patterns
- [Synthesis Protocol](references/synthesis-protocol.md) - Multi-agent panel + consensus protocol
- [Evolution Scoring](references/evolution-scoring.md) - Timelessness scoring rubric
- [Architecture & Config](references/architecture-and-config.md) - Pattern selection + aspirational config block
- [Changelog](references/changelog.md) - Full version history

## Related Skills

| Skill | Relationship |
|-------|--------------|
| skill-composer | Can orchestrate created skills |
| claude-authoring-guide | Deeper patterns reference |
| codereview | Pattern for multi-agent panels |
| maker-framework | Zero error standard source |

## Extension Points

1. **Additional lenses:** add to `references/multi-lens-framework.md`
2. **New synthesis agents:** extend the panel beyond 4 in `references/synthesis-protocol.md`
3. **Custom architecture patterns:** add to `references/architecture-and-config.md`
4. **Script patterns:** add to `references/script-patterns-catalog.md`
5. **Machine setup:** this skill is not vendored (`.gitignore` excludes `skills/skillforge/`) — clone the fork, not upstream: `git clone https://github.com/xiaofjiang/SkillForge ~/.claude/skills/skillforge` (AUDIT-049). Fork exists so local edits (like this trim, and the `discover_skills.py` case-insensitivity fix) persist across machines instead of living only in one untracked clone.
