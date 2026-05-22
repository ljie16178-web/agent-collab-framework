# Installation Guide

## Create Instance Binding

Create `.roo/skills/skilldistiller_instance.md`:

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

## Trigger After Collaboration

After tri-role loop completes and user rates ≥ 4/5:

Say: "蒸馏当前项目技能"

## Meta-Skill Update

When patterns validated in ≥ 2 projects, confirm update to `auto_prompt_3agents`.
