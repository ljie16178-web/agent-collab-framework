# Installation Guide

## Copy Framework to New Project

```bash
# From existing project with auto_prompt_3agents
cp -r skills/auto_prompt_3agents/ new-project/skills/
cp -r skills/skilldistiller/ new-project/skills/
```

## Create Instance Binding

Create `new-project/.roo/skills/auto_prompt_instance.md`:

```markdown
# AutoPrompt Instance — NewProject

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
| `{{PROJECT_NAME}}` | new-project |
| `{{PROJECT_DOMAIN}}` | your domain |
```

## Create Rules Directory

Create `new-project/.roo/rules/` with:
- `rules.md` — Memory management rules
- `safety-protocol.txt` — Security and behavior protocol
- `self-improvement-triggers.md` — Auto-learning triggers

## Verify Installation

Say: "生成项目目前的专属提示词"

Expected: Framework loads instance bindings → generates task sheet with your roles.
