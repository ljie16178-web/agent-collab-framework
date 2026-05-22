# AutoPrompt_3Agent — Tri-Role Prompt Generator

## Overview

Framework-level skill for generating standardized tri-role (Designer-Coder-Reviewer) collaboration task sheets. Cross-project reusable — role bindings are injected via `.roo/skills/auto_prompt_instance.md`.

## Architecture

```
skills/auto_prompt_3agents/SKILL.md          ← FRAMEWORK (this file)
.roo/skills/auto_prompt_instance.md          ← INSTANCE (project binding)
.roo/rules/*.{md,txt}                         ← Rules (auto-injected)
```

## Trigger

Say any of:
- "生成项目目前的专属提示词"
- "生成项目专属提示词" / "auto prompt"
- "生成三智能体提示词"

## Instance Binding

Create `.roo/skills/auto_prompt_instance.md` in each new project:

```markdown
# AutoPrompt Instance — YourProject

| Variable | Value |
|------|------|
| `{{DESIGNER_NAME}}` | YourDesignerName |
| `{{DESIGNER_MODE}}` | architect |
| `{{DESIGNER_MODEL}}` | Kimi |
| `{{CODER_NAME}}` | YourCoderName |
| `{{CODER_MODE}}` | code |
| `{{CODER_MODEL}}` | DeepSeek |
| `{{REVIEWER_NAME}}` | YourReviewerName |
| `{{REVIEWER_MODE}}` | ask |
| `{{REVIEWER_MODEL}}` | GLM |
| `{{PROJECT_NAME}}` | your-project |
| `{{PROJECT_DOMAIN}}` | your domain |
```

## Features

- Confidence check & proactive inquiry (≥ 90% → direct output, < 70% → confirm)
- State reset command (clean context between rounds)
- Loop correction (review → fix → re-review until pass)
- Deliverable solidification (extract reusable templates after completion)

## Integration

- **Memos MCP**: Auto `search_memory` / `add_message`
- **SkillDistiller**: Auto-trigger after loop completion for pattern distillation
- **Self-Improving-Agent**: Auto-log errors and corrections

## Version

v1.0 — Framework abstraction with `{{variable}}` bindings
