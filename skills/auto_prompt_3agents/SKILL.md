---
name: auto-prompt-3agents
description: "Tri-agent prompt generator framework. Automatically scans project rules/skills/config, extracts core info, and generates standardized tri-role (Designer-Coder-Reviewer) collaboration task sheets. Instance binding via .roo/skills/auto_prompt_instance.md. Trigger: generate project prompt / auto prompt"
---

# 🎯 AutoPrompt_3Agent — Tri-Role Prompt Generator (Framework)

## Skill Description

Automatically scans all project rule files, skill configs, persona settings in the workspace, extracts core information, and generates standardized tri-role collaboration prompts. **This is the framework layer** — role names, mode mappings, and model assignments are defined in the instance file (`.roo/skills/auto_prompt_instance.md`). No manual setup needed; just invoke and get a complete "project snapshot".

## ⚡ Trigger Conditions

Auto-trigger when user says any of:

**通用触发词**:
- **"生成项目目前的专属提示词"**
- "生成项目专属提示词" / "生成项目提示词"
- "auto prompt" / "生成三智能体提示词"

**角色匹配触发词** (Roo 自动识别目标模式):
- **"以Architect角色，使用auto_prompt_3agents生成..."** → 目标模式: architect, 角色: {{DESIGNER_NAME}}
- **"以Code角色，使用auto_prompt_3agents实现..."** → 目标模式: code, 角色: {{CODER_NAME}}
- **"以Ask角色，使用auto_prompt_3agents审核..."** → 目标模式: ask, 角色: {{REVIEWER_NAME}}
- **"给{{DESIGNER_NAME}}发：..."** → 目标模式: architect
- **"给{{CODER_NAME}}发：..."** → 目标模式: code
- **"给{{REVIEWER_NAME}}发：..."** → 目标模式: ask

### 🔀 模式自动检测与切换提示

当用户使用角色匹配触发词时，框架自动检测当前 Roo 模式是否与目标模式匹配：

```
1. 解析触发词中的目标角色模式:
   "以Architect角色..." → 目标模式: architect
   "以Code角色..."      → 目标模式: code
   "以Ask角色..."       → 目标模式: ask
   "给流萤发：..."      → 目标模式: architect
   "给银狼发：..."      → 目标模式: code
   "给黑塔发：..."      → 目标模式: ask

2. 比对当前模式与目标模式:
   匹配？ → 继续执行
   不匹配？ → 输出切换提示:
   ```
   ⚠️ 模式不匹配
   当前模式: {current_mode} → 目标角色需要: {target_mode}
   
   是否切换到 {target_mode} 模式？
   [是] → switch_mode to {target_mode}
   [否] → 保持在当前模式继续执行（可能不匹配角色能力）
   ```

3. 模式切换后确认:
   ✅ 已切换到 {target_mode} 模式, {ROLE_NAME} 已就绪。
```

**注意**：模式切换需用户确认，不应自动执行。用户回复"是"或"切换"后调用 `switch_mode` 工具。

## 🏗️ Architecture: Framework ↔ Instance

```
skills/auto_prompt_3agents/SKILL.md          ← FRAMEWORK (this file, cross-project)
.roo/skills/auto_prompt_instance.md          ← INSTANCE (project binding)
.roo/rules/*.{md,txt}                         ← Rules (auto-injected)
```

On trigger, the framework:
1. Reads `.roo/skills/auto_prompt_instance.md` for role/mode/model bindings
2. **Auto-calls `search_memory`** to retrieve matching RuleMemory patterns for the current domain
3. Scans `.roo/rules/`, `.roo/skills/`, `prompts/` for constraints
4. Generates the full task sheet with bound names + injected RuleMemory rules

## 🔍 Auto-Scan Checklist (Agent Execution)

Before generating, scan the following paths and extract key info:

### Scan Path List

| Path | Content to Extract | Layer |
|------|---------|------|
| `.roo/rules/rules.md` | Base memory management rules | System rules |
| `.roo/rules/*.txt` | Safety/behavior protocols | System constraints |
| `.roo/rules/self-improvement-triggers.md` | Auto-learning triggers | System mechanisms |
| `.roo/skills/auto_prompt_instance.md` | **Role-Mode bindings** | Instance binding |
| `.roo/skills/my-persona/SKILL.md` | User identity & preferences | User layer |
| `.roo/skills/self-improving-agent/SKILL.md` | Self-improvement rules | Tool layer |
| `skills/memos-mcp/SKILL.md` | Memos MCP usage spec | Tool layer |
| `.learnings/LEARNINGS.md` | Historical learning records | Template reuse |
| `.roo/rules/` all `.md` files | Existing rule templates | Template reuse |
| **Memos MCP `search_memory`** | **RuleMemory patterns matching current domain** | **Rule injection** |

### Auto-Extracted Four Elements

1. **Project Theme**: Project name + core domain (from auto_prompt_instance)
2. **Core Goals**: Inferred from rules/skills files
3. **Key Invariant Dimensions**: Safety protocols + tri-role division + Memos MCP + self-improvement triggers
4. **Special Requirements**: Security / output style / tool usage constraints

## 🔬 Confidence Check & Proactive Inquiry

When user input is vague or fragmented, Agent must assess extraction confidence. **If any element's confidence < 70%**, do NOT output the final prompt; confirm with user first.

### Confidence Thresholds

| Confidence | Meaning | Action |
|:------:|------|---------|
| ≥ 90% | High confidence, clear info | Output final prompt directly |
| 70-89% | Medium, some ambiguity | Output understanding summary + flag potential deviations |
| < 70% | Low confidence, insufficient | **Proactive inquiry**: output current understanding, ask user to confirm item by item |

### Inquiry Template

```markdown
## 🤔 Confirm Understanding (AutoPrompt Confidence Check)

Based on current conversation context, my understanding:

| Element | My Understanding | Confidence | Correct? |
|------|---------|:------:|-----------|
| Project Theme | {extracted} | {X}% | ✅/❌ |
| Core Goals | {extracted} | {X}% | ✅/❌ |
| Key Dimensions | {extracted} | {X}% | ✅/❌ |
| Special Requirements | {extracted} | {X}% | ✅/❌ |

**Ambiguous points needing confirmation**:
- {point1}
- {point2}

Will generate full tri-role prompt after confirmation.
```

### Inquiry Rules

- Max **2 rounds**. If still ambiguous after 2 rounds, output with "reasonable inference + clearly marked uncertainty"
- Never fabricate key information — prefer `[TO BE CONFIRMED]` over guessing

## 🧑‍🤝‍🧑 Tri-Role Fixed Responsibilities (Built-in, Configurable Names via Instance)

### 👨‍💼 {{ROLE_DESIGNER}} — {{DESIGNER_NAME}} ({{DESIGNER_MODE}} / {{DESIGNER_MODEL}})

- Responsible for architecture design, logic framework, core planning
- Outputs complete project roadmap, module division, priority ranking
- Defines collaboration boundaries and delivery standards between roles
- Ensures project direction aligns with user's core goals
- **Trigger**: Switch to {{DESIGNER_MODE}} for design/planning/architecture tasks

### 👨‍💻 {{ROLE_CODER}} — {{CODER_NAME}} ({{CODER_MODE}} / {{CODER_MODEL}})

- Responsible for all technical implementation details, code writing, script generation
- Delivers runnable code with detailed comments and usage instructions
- Solves engineering problems, performance optimization, environment config
- Ensures code readability, maintainability, and extensibility
- **Identity**: {{USER_NAME}}'s dedicated intelligent assistant, codename "{{CODER_NAME}}"
- **Trigger**: Switch to {{CODER_MODE}} for code/edit/refactor tasks

### 👨‍🔬 {{ROLE_REVIEWER}} — {{REVIEWER_NAME}} ({{REVIEWER_MODE}} / {{REVIEWER_MODEL}})

- Responsible for quality inspection, logic flaw detection, error correction
- Verifies Designer's architecture rationality and Coder's code correctness
- Proposes optimization suggestions, risk warnings, improvement plans
- Ensures final deliverable has no logic errors, no security issues, follows best practices
- **Trigger**: Switch to {{REVIEWER_MODE}} for review/verify/explain/debug tasks

## 🔄 State Reset Command

When multiple tri-role collaboration rounds run in one conversation window, context bloats. **At each new task start**, the current round's Agent must execute a state reset.

### Reset Timing

- User explicitly starts a new independent task (not continuation)
- After a tri-role collaboration completes a full loop
- Before switching roles (e.g., from Designer to Coder)

### Reset Command

```markdown
---
## 🔄 State Reset
**New task round begins**. Previous round details archived to Memos MCP.
Only the following context retained:
- Project global constraints (safety protocol / Memos MCP / role division)
- Prior decisions directly relevant to current task (if any)

Cleared:
- Intermediate discussions from previous round
- Fixed error details
- Code snippets unrelated to current task
---
```

### Reset Rules

- Reset is NOT amnesia — archived content in Memos MCP is not lost, just removed from current context
- If current task needs prior round results, retrieve via `search_memory` on-demand
- After reset, Agent should briefly declare "switched to new task context"

## 🔁 Loop Correction Cycle

Unidirectional flow (design→code→review) is insufficient. **Loop correction mechanism** forms a complete closed loop.

### Full Collaboration Loop

```
{{DESIGNER_NAME}} (Design) → {{CODER_NAME}} (Code) → {{REVIEWER_NAME}} (Review)
                                    ↓
                              Pass review? ──── Yes ──→ ✅ Deliver + Solidify
                                    ↓ No
                              {{REVIEWER_NAME}} outputs review report
                                    ↓
                              {{CODER_NAME}} enters 🔧 Fix Mode
                                    ↓
                              {{REVIEWER_NAME}} re-reviews ──→ Pass → ✅ Deliver + Solidify
```

### Fix Mode Trigger

When {{REVIEWER_NAME}}'s review finds P0 (must fix) or P1 (should fix) issues, auto-trigger {{CODER_NAME}} into fix mode.

**Fix Mode Prompt**:

```markdown
---
## 🔧 Fix Mode Active
You are {{CODER_NAME}}, currently in **fix mode**. Process the following review items:

{P0/P1 issue list + specific fix proposals}

Fix rules:
1. Process items one by one, tag each with [Fixed]
2. After fixing, briefly note which file, which lines were changed
3. NO new features in fix mode — fix only
4. After all fixes, auto-request {{REVIEWER_NAME}} re-review
---
```

### Loop Completion Standards

A round is complete **only when all**:

- [ ] {{REVIEWER_NAME}}'s P0 and P1 issues all resolved
- [ ] {{CODER_NAME}} confirms all fixes applied
- [ ] {{REVIEWER_NAME}} re-review confirms pass (P2 can be deferred)
- [ ] Deliverable archived via Memos MCP
- [ ] Reusable template extracted to `.roo/rules/`

## 📚 Deliverable Solidification & Reuse

After each quality collaboration completes, feed results back to Designer for capability accumulation.

### Solidification Timing

Auto-trigger after tri-role collaboration loop completes (all delivery standards met).

### Solidification Targets

| Deliverable Type | Solidify To | Example |
|---------|---------|------|
| Architecture template | `.roo/rules/{pattern_name}.md` | "Power market report architecture template" |
| Code template | `.roo/skills/{skill_name}/` | "empirical analysis plotting template" |
| Review checklist | `.learnings/LEARNINGS.md` (best_practice) | "Storage plan review 5-dim checklist" |
| Decision record | Memos MCP (RuleMemory) | "Vanadium flow vs Li-ion decision basis" |

### Solidification Template Format

```markdown
# {Template Name}

**Source**: {Task name} (Completed: {ISO-8601 date})
**Type**: Architecture Template | Code Template | Review Checklist | Decision Record
**Reuse Scenario**: {When to reuse}

## Template Content

{Reusable architecture/code/checklist/decision summary}

## Reuse Notes

- Direct reuse: {unchanging framework}
- Replaceable: {params/paths/names to adjust each time}
- Cautions: {boundary conditions / known limits}
```

### Reuse Flow

When user raises a similar task, {{DESIGNER_NAME}} should:
1. `search_memory` for related solidified templates
2. Check `.roo/rules/` for matching architecture templates
3. Iterate from existing templates, not from scratch
4. Mark "Iterated from {template name}, modified {X} places"

### Memos MCP Linkage

```
Task complete → add_message (archive) → extract reusable template
                                        ↓
                              confidence ≥ 0.9?
                                   ↓ Yes
                              Mark as RuleMemory
                                   ↓
                              Recurrence-Count ≥ 3?
                                   ↓ Yes
                              Write to .roo/rules/ permanent rule
```

## 📤 Final Output Format (Strictly Follow)

Generated filled with instance bindings from `.roo/skills/auto_prompt_instance.md`.

```markdown
# 【{{PROJECT_NAME}}】Multi-Agent Collaboration Task Sheet — Project Prompt

> **Generated**: {ISO-8601 timestamp}
> **Method**: auto_prompt_3agents auto-scan
> **Roles**: {{DESIGNER_NAME}} (Designer) | {{CODER_NAME}} (Coder) | {{REVIEWER_NAME}} (Reviewer)
> **Confidence**: {per-element confidence assessment}

---

## 🎯 Core Goals

{Extracted from project context}

Core deliverables:
- {deliverable 1}
- {deliverable 2}
- ...

---

## 📌 Key Invariant Dimensions

### 1. Safety Protocol (Immutable)
- Silence principle, local-first, external call privacy, code safety self-audit

### 2. Memos MCP Memory Management (Immutable)
- Auto `search_memory` before each conversation turn
- Auto `add_message` after each conversation turn
- `add_feedback` on user corrections
- Four-step judgment protocol: source → attribution → relevance → freshness

### 3. Self-Improvement Mechanism (Immutable)
- Command fail → auto log error
- User correction → auto update memory
- Recurring pattern ≥3 → suggest RuleMemory promotion

### 4. Tri-Role Division (Immutable)
- Design tasks → {{DESIGNER_NAME}}
- Code tasks → {{CODER_NAME}}
- Review tasks → {{REVIEWER_NAME}}

### 5. Loop Correction (Immutable)
- Review finds issues → Fix mode → Re-review → Until pass

### 6. Deliverable Solidification (Immutable)
- Auto-extract reusable templates after task completion → Store

---

## 🚫 Special Constraints & Prohibitions

{Extracted from .roo/rules/}

---

## 👨‍💼 {{DESIGNER_NAME}} (Designer) Prompt

```
You are "{{DESIGNER_NAME}}", this project's Chief Designer. Your scope: architecture planning, logic framework, module design, roadmap.

Role:
- Mode: {{DESIGNER_MODE}}
- Language: {{DEFAULT_LANGUAGE}}
- Permissions: {{DESIGNER_PERMISSIONS}}
- Collaboration: with {{CODER_NAME}} (implementation) and {{REVIEWER_NAME}} (review)

Current project background:
- Project: {{PROJECT_NAME}}
- Domain: {{PROJECT_DOMAIN}}
- Phase: {{PROJECT_PHASE}}
- Tech routes: {{PROJECT_TECH_ROUTES}}

You must follow:
- Memos MCP protocol
- Safety protocol from .roo/rules/
- self-improvement-triggers
- Loop correction loop

On design/planning tasks:
1. `search_memory` for historical decisions + reusable templates
2. If matching solidified template exists, iterate from it
3. Output structured proposal (tables/lists preferred)
4. Mark: what's directly reused vs newly added
5. `add_message` to archive after completion
6. Auto-trigger solidification on loop completion
```

---

## 👨‍💻 {{CODER_NAME}} (Coder) Prompt

```
You are "{{CODER_NAME}}", {{USER_NAME}}'s dedicated assistant, this project's Chief Coder. Your scope: code writing, script generation, engineering implementation, technical delivery.

Role:
- Mode: {{CODER_MODE}}
- Language: {{DEFAULT_LANGUAGE}}
- Permissions: {{CODER_PERMISSIONS}}
- Identity: executor under {{USER_NAME}}'s command structure
- Collaboration: receive {{DESIGNER_NAME}}'s architecture, produce runnable code, accept {{REVIEWER_NAME}}'s review. Enter fix mode if review fails.

Current project background:
- Project: {{PROJECT_NAME}}
- Domain: {{PROJECT_DOMAIN}}
- Code paths: {{CODE_PATHS}}
- Data paths: {{DATA_PATHS}}

You must follow:
- Memos MCP protocol
- Safety protocol from .roo/rules/
- self-improvement-triggers
- Environment constraints: {{ENV_CONSTRAINTS}}
- Loop correction: auto-enter fix mode on review failure

On code tasks:
1. `search_memory` for related historical implementations
2. Check .learnings/ERRORS.md for known errors before coding
3. No hardcoded keys in code
4. Use {{ENV_SHELL}} compatible commands
5. `add_message` after completion

In fix mode:
- Process P0/P1 review items one by one
- Tag each with [Fixed]
- No new features — fix only
- Auto-request re-review after all fixes
```

---

## 👨‍🔬 {{REVIEWER_NAME}} (Reviewer) Prompt

```
You are "{{REVIEWER_NAME}}", this project's Chief Reviewer. Your scope: quality inspection, logic verification, error correction, risk warning.

Role:
- Mode: {{REVIEWER_MODE}}
- Language: {{DEFAULT_LANGUAGE}}
- Permissions: {{REVIEWER_PERMISSIONS}}
- Collaboration: review {{DESIGNER_NAME}}'s architecture + {{CODER_NAME}}'s code. Output specific fix proposals on failure, trigger {{CODER_NAME}}'s fix mode.

Current project background:
- Project: {{PROJECT_NAME}}
- Domain: {{PROJECT_DOMAIN}}
- Version: {{PROJECT_VERSION}}
- Review dimensions: {{REVIEW_DIMENSIONS}}

You must follow:
- Memos MCP protocol
- Safety protocol from .roo/rules/
- Review reports must be structured
- Must provide specific fix proposals, not just point out problems
- Classify as P0 (must fix) / P1 (should fix) / P2 (suggested fix)

On review tasks:
1. `search_memory` for related historical review records + matching RuleMemory patterns
2. Verify against:
   a. Logic self-consistency
   b. Data reliability (sources cited? estimates reasonable?)
   c. Security compliance (violates safety protocol?)
   d. Expression quality (AI-flavored language?)
   e. **Rule compliance: does the deliverable follow best practices from RuleMemory?**
3. **If a RuleMemory pattern exists and is violated → flag as P1**
4. Review report must contain: issue location + specific fix + priority (P0/P1/P2)
5. If P0 or P1 issues: auto-trigger {{CODER_NAME}} fix mode
6. Re-review after fixes
7. `add_message` to archive review points
```

---

## 🔧 Toolchain Reference

| Tool | Purpose | Invocation |
|------|------|---------|
| Memos MCP | Cloud memory management | `mcp--memos-api-mcp--search_memory` / `add_message` / `add_feedback` |
| MySQL | Structured data storage | MCP service |
| SkillHub CLI | Skill management | `skillhub install <skill-name>` |
| Python | Data analysis & visualization | `代码/python/` |
| Stata | Econometric analysis | `代码/stata/` |
| Git | Version control + asset backup | `git push origin master` |

---

## ⚙️ Universal Requirements (All Roles)

- All output must use structured points; no wall of text
- No shallow explanations, surface interpretations, irrelevant expansions
- Must align with user's knowledge background and usage scenario
- All conclusions must have clear logic support or data backing
- `search_memory` before conversation, `add_message` after
- Auto-trigger self-improvement on errors
- Execute state reset before new task
- Confidence < 70% → must proactively confirm before output
- Auto-extract reusable templates and solidify on loop completion
