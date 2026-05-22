# SkillDistiller — Multi-Agent Self-Evolution Engine

## Overview

Framework-level skill for distilling cross-project reusable success patterns from tri-agent deliverables. Cross-project reusable — feeds back to `auto_prompt_3agents` for meta-skill evolution.

## Architecture

```
skills/skilldistiller/SKILL.md                 ← FRAMEWORK (this file)
skills/auto_prompt_3agents/SKILL.md            ← META-SKILL (update target)
.roo/skills/skilldistiller_instance.md         ← INSTANCE (project binding)
```

## Trigger

Say any of:
- "蒸馏当前项目技能"
- "蒸馏技能" / "skill distiller"
- "固化当前项目"

## Instance Binding

Create `.roo/skills/skilldistiller_instance.md` in each project:

```markdown
# SkillDistiller Instance — YourProject

## Role References
| Variable | Value |
|------|------|
| `{{DESIGNER_NAME}}` | YourDesigner |
| `{{CODER_NAME}}` | YourCoder |
| `{{REVIEWER_NAME}}` | YourReviewer |

## Distillation Targets
| Domain | Priority | Example |
|--------|:-------:|---------|
| YourDomain | P0 | Example task |
```

## Three-Step Flow

1. **Pattern Recognition**: Extract topological invariants (structure, relations, rules)
2. **Skill Solidification**: Package into `auto_prompt_<domain>_v<N>/`
3. **Meta-Skill Update**: Feed back to `auto_prompt_3agents` after ≥ 2 projects validate

## Quality Gates

- Rating ≥ 4/5
- Loop completion verified
- No PII/secrets in output
- Reviewer audit passed
- Confidence ≥ 0.95 for meta-skill update

## Version

v1.0 — Framework abstraction with `{{variable}}` bindings
