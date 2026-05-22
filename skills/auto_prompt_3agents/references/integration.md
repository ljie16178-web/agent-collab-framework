# Integration Guide

## For New Projects

### Step 1: Copy Framework

Copy `skills/auto_prompt_3agents/` and `skills/skilldistiller/` to your new project.

### Step 2: Create Instance Binding

Create `.roo/skills/auto_prompt_instance.md`:

```markdown
# AutoPrompt Instance — YourProject

| Variable | Value |
|------|------|
| `{{DESIGNER_NAME}}` | YourDesigner |
| `{{DESIGNER_MODE}}` | architect |
| `{{DESIGNER_MODEL}}` | Kimi |
| `{{CODER_NAME}}` | YourCoder |
| `{{CODER_MODE}}` | code |
| `{{CODER_MODEL}}` | DeepSeek |
| `{{REVIEWER_NAME}}` | YourReviewer |
| `{{REVIEWER_MODE}}` | ask |
| `{{REVIEWER_MODEL}}` | GLM |
| `{{PROJECT_NAME}}` | your-project |
| `{{PROJECT_DOMAIN}}` | your domain |
```

### Step 3: Create Rules

Create `.roo/rules/` with your project's safety protocols and memory management rules.

### Step 4: Trigger

Say: "生成项目目前的专属提示词" → Framework loads instance bindings → Generates task sheet.

## Integration with SkillDistiller

After a full collaboration loop completes:
1. Say: "蒸馏当前项目技能"
2. SkillDistiller extracts patterns from deliverables
3. Generates `skills/auto_prompt_<domain>_v<N>/`
4. After ≥ 2 projects validate, auto-updates this framework

## Memos MCP Linkage

```
Trigger → search_memory (historical context)
        → scan .roo/rules/ + .roo/skills/ + prompts/
        → inject instance bindings → generate
        → add_message (archive)
```
