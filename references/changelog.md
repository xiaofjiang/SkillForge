# Changelog

Version history, moved out of SKILL.md during the 2026-07 trim (System-Audit AUDIT-028) to keep the main file to its used core. Current version lives in SKILL.md's frontmatter (`metadata.version`).

### v3.2.0
- Added Script Integration Framework for agentic skills
- Added 4th Script Agent to synthesis panel (conditional)
- Added Phase 1D: Automation Analysis
- Added Automation Lens questions to regression questioning
- Created `references/script-integration-framework.md`
- Created `references/script-patterns-catalog.md`
- Created `assets/templates/script-template.py`
- Updated skill-spec-template.xml with `<scripts>` section
- Updated validate-skill.py with script validation
- Skills can now include self-verifying Python scripts

### v3.1.0
- Added progressive disclosure structure
- Fixed frontmatter for packaging compatibility
- Added validation & packaging section
- Deep dive sections now collapsible

### v3.0.0
- Complete redesign as ultimate meta-skill
- Added regression questioning loop
- Added multi-lens analysis framework (11 models)
- Added evolution/timelessness core lens
- Added multi-agent synthesis panel

### v2.0.0
- Pattern selection guide
- Quality standards checklist

### v1.0.0
- Basic skill structure

### 2026-07 (post-fork, local)
- Forked from `tripleyak/SkillForge` to `xiaofjiang/SkillForge` (AUDIT-049) so local edits persist across machines instead of living only in an untracked clone.
- Trimmed SKILL.md from 871 to under 350 lines (AUDIT-028): kept Phase-0 triage, the create/improve decision, the spec+generation workflow, and output conventions inline; moved the 11 thinking lenses, multi-agent synthesis panel, consensus protocol, temporal-projection scoring, architecture pattern selection, the configuration block, and this changelog to `references/`.
- Preserved a pre-existing local fix in `scripts/discover_skills.py`: case-insensitive `SKILL.md` matching (the simple-pattern glob branch was silently dropping every uppercase `SKILL.md` on case-sensitive filesystems).
