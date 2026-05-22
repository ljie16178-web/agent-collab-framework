---
name: skill-distiller
description: "Multi-agent system self-evolution core engine. Auto-parses tri-agent deliverables and user ratings, identifies cross-project reusable success patterns, solidifies them into callable vertical skills, and feeds universal best practices back to AutoPrompt_3Agent. Trigger: distill project / 蒸馏当前项目技能"
---

# 🧪 SkillDistiller — Multi-Agent Self-Evolution Core Engine (Framework)

## Skill Description

The self-evolution core engine of the multi-agent system. Automatically parses complete tri-agent project deliverables and user final ratings, identifies cross-project reusable success patterns, solidifies them into directly callable vertical skills, and feeds universal best practices back to the meta-skill `auto_prompt_3agents`. Implements the positive cycle: **"Single success → Reusable capability → System-wide upgrade"**.

**This is the framework layer** — role names and mode bindings are loaded from `.roo/skills/auto_prompt_instance.md`.

## ⚡ Trigger Conditions

Auto-trigger when user says any of:
- **"蒸馏当前项目技能"**
- "蒸馏技能" / "skill distiller"
- "固化当前项目" / "提取项目模式"

## 📥 Input Format (Auto-recognized, No Manual Fill)

1. **Complete tri-agent project deliverables**:
   - {{DESIGNER_NAME}} output: architecture docs, module division, roadmap
   - {{CODER_NAME}} output: complete code, comments, env config, usage instructions
   - {{REVIEWER_NAME}} output: review report, issue list, optimization proposals
2. **User final rating**: project success score (1-5) + core highlights
3. **Trigger command**: `蒸馏当前项目技能`

## ⚙️ Core Execution Flow (Strict 3 Steps)

### 🔍 Step 1: Pattern Recognition (Extract Topological Invariants)

**Core principle**: Only extract **structure, relations, rules** — completely filter project-specific content and data.

Extract role-specific pattern libraries:

#### {{DESIGNER_NAME}} Pattern Library

1. **Architecture template library**: Extract validated universal module division, hierarchy structure, data flow topology
2. **Invariant dimension library**: Extract core problem dimensions that must be covered, key constraint conditions
3. **Task division template**: Extract optimal collaboration boundaries between tri-roles, delivery standards, priority ranking
4. **Goal decomposition template**: Extract logic for breaking fuzzy user needs into actionable tasks

#### {{CODER_NAME}} Pattern Library

1. **Utility function library**: Extract reusable core utility functions, data structures, algorithm implementations
2. **Code template library**: Extract standardized code structure, comment conventions, error handling patterns
3. **Env config template**: Extract unified dependency lists, version requirements, deployment scripts
4. **Debug guide template**: Extract common troubleshooting flows, performance optimization methods

#### {{REVIEWER_NAME}} Pattern Library

1. **Checklist library**: Extract logic points, security vulnerabilities, boundary conditions that must be checked
2. **Risk warning library**: Extract common failure modes, potential risk points
3. **Optimization proposal template**: Extract universal performance optimization, maintainability improvement, scalability enhancement methods

**Quality Control**:
- Only distill projects with user rating ≥ 4/5
- Mark validation count per pattern; only retain patterns with ≥ 1 validation
- {{REVIEWER_NAME}} reviews all extracted patterns for logic errors

### 🧱 Step 2: Solidify Skill (Generate Vertical Domain Skill)

Package all extracted patterns from Step 1 into a complete, independently callable vertical skill file.

**Naming**: `auto_prompt_<domain>_v<version>`

**Output format**: Fully compatible with `auto_prompt_3agents` framework

**Generated content**:
1. Skill description: application scenario, core capability, problem solved
2. Built-in pattern library: embed all role-specific patterns from Step 1
3. Default invariant dimensions: auto-fill core dimensions for this project type
4. Default constraints: auto-fill universal constraints and best practices
5. Tri-role default tasks: auto-fill standard role assignments and task checklists

**Storage**: `skills/auto_prompt_<domain>_v<version>/` + instance in `.roo/skills/`

### 🔄 Step 3: Meta-Skill Update (Feedback to AutoPrompt_3Agent)

Write back cross-project validated success patterns to `auto_prompt_3agents`:

1. Identify patterns appearing in ≥ 2 projects
2. Update corresponding sections in `auto_prompt_3agents` (invariant dimensions / constraints / role templates)
3. Tag update as "Distilled via SkillDistiller"

## 📤 Output Format

```markdown
# 🧪 Skill Distillation Report v{version}

## 📊 Project Basic Info
- **Project Name**: {auto-extracted}
- **Project Rating**: {auto-extracted} (1-5)
- **Core Highlights**: {auto-extracted}
- **Distillation Time**: {ISO-8601 timestamp}

## 🔍 Extracted Success Patterns

### {{DESIGNER_NAME}} Patterns (X total)
- {Pattern name}: {description}
   - Validations: {n} | Reuse scenario: {when}

### {{CODER_NAME}} Patterns (X total)
- {Pattern name}: {description}
   - Validations: {n} | Reuse scenario: {when}

### {{REVIEWER_NAME}} Patterns (X total)
- {Pattern name}: {description}
   - Validations: {n} | Reuse scenario: {when}

## 🧱 Generated Vertical Skill
- ✅ Generated: `auto_prompt_<domain>_v<version>` → `skills/auto_prompt_<domain>_v<version>/SKILL.md`

## 🔄 Meta-Skill Update Suggestions
Patterns validated in ≥ 2 projects, suggest updating `auto_prompt_3agents`:

| Pattern | Validations | Target Section | Suggested Content |
|---------|:----------:|---------|---------|
| {name} | {n} | {section} | {content} |

### ⚙️ Confirm
Execute meta-skill update? [Yes/No]
```

## 🔗 Infrastructure Integration

### Memos MCP Linkage

```
Project complete → add_message (archive full deliverables)
                 → SkillDistiller triggers
                 → Extract patterns → Solidify vertical skill
                 → add_message (archive distillation report + pattern_key)
                 → Cross-project validation ≥ 2 → Mark as RuleMemory
```

**After each distillation, auto-call for every extracted pattern**:

```
mcp--memos-api-mcp--add_message:
  conversation_first_message: <first user message>
  messages: [{
    role: "assistant",
    content: "[SkillDistiller] pattern_key: {domain}.{category}.{desc} | "
             "标题: {pattern_title} | 验证次数: {n} | "
             "来源项目: {PROJECT_NAME} | confidence: {rating/5.0}"
  }]
```
> Required: `memory_type: RuleMemory`, `source: skill_distillation`, `tags: ["skill_distillation", "{domain}"]`

### AutoPrompt_3Agent Closed Loop

```
auto_prompt_3agents (Generate task sheet)
        ↓
Tri-role collaboration executes
        ↓
Loop completion + user rating
        ↓
SkillDistiller (Distill patterns)
        ↓
auto_prompt_<domain>_v<N> (Vertical skill)
        ↓
Cross-project validation ≥ 2
        ↓
Write back to auto_prompt_3agents (Meta-skill evolution)
```

### Self-Improving-Agent Linkage

- High-value patterns in distillation report → `.learnings/LEARNINGS.md` (category: best_practice)
- Cross-project validation ≥ 3 → Trigger RuleMemory promotion
- Write to `.roo/rules/skill_<domain>.md` permanent rule

### Solidification Complementarity

| Mechanism | Scope | Output |
|------|:-----:|------|
| auto_prompt_3agents solidification | Single project | `.roo/rules/` rule templates |
| SkillDistiller distillation | Cross-project | `skills/auto_prompt_<domain>_v<N>/` vertical skill + meta-skill update |

## 🔒 Quality Gates

- **Minimum rating**: Only distill projects rated ≥ 4/5
- **Loop verified**: Only distill projects that completed the full loop (design→code→review→fix→re-review→pass)
- **Security audit**: Distillation reports must NOT contain keys, passwords, PII
- **{{REVIEWER_NAME}} review**: All extracted patterns must pass {{REVIEWER_NAME}}'s logic audit
- **Confidence requirement**: Patterns distilled for meta-skill update require confidence ≥ 0.95

## 📊 Distillation Stats Panel

Auto-updated after each distillation:

| Stat | Current |
|------|:-----:|
| Total Projects Distilled | {n} |
| Vertical Skills Generated | {n} |
| Meta-Skill Updates | {n} |
| Pattern Library Size | {n} patterns |
| Avg Project Rating | {x.x}/5 |

## 🔧 Tool Dependencies

| Tool | Purpose |
|------|------|
| Memos MCP | Archive distillation reports + cross-project pattern validation |
| `.roo/skills/self-improving-agent/` | Distillation patterns → best_practice records |
| `skills/auto_prompt_3agents/` | Meta-skill update target |
| `.roo/rules/` | Cross-project permanent rule storage |
| `.learnings/` | Local pattern validation records |
