# SkillDistiller Integration Guide

## For New Projects

### Step 1: Create Instance Binding

Create `.roo/skills/skilldistiller_instance.md`:

```markdown
# SkillDistiller Instance — YourProject

## Role References (from auto_prompt_instance.md)
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

### Step 2: Trigger After Completion

After tri-role collaboration completes and user rates ≥ 4/5:

Say: "蒸馏当前项目技能"

### Step 3: Confirm Meta-Skill Update

If patterns validated in ≥ 2 projects:

Framework suggests updating `auto_prompt_3agents`. Confirm to evolve the meta-skill.

## Closed Loop

```
auto_prompt_3agents (Generate)
    ↓
Collaboration + Rating
    ↓
SkillDistiller (Distill)
    ↓
auto_prompt_<domain>_v<N>
    ↓
Cross-project validation ≥ 2
    ↓
Update auto_prompt_3agents (Evolve)
```

## Memos MCP Linkage

```
Trigger → add_message (archive deliverables)
        → Extract patterns → solidify skill
        → add_message (archive report + pattern_key)
        → Cross-project validation → RuleMemory
```
