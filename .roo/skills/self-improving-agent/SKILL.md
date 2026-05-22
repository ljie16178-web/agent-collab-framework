---
name: self-improvement
description: "Captures learnings, errors, and corrections to enable continuous improvement. Use when: (1) A command or operation fails unexpectedly, (2) User corrects Claude ('No, that's wrong...', 'Actually...'), (3) User requests a capability that doesn't exist, (4) An external API or tool fails, (5) Claude realizes its knowledge is outdated or incorrect, (6) A better approach is discovered for a recurring task. Also review learnings before major tasks."
metadata:
---

# Self-Improvement Skill

Log learnings and errors to markdown files for continuous improvement. Coding agents can later process these into fixes, and important learnings get promoted to project memory.

When Memos MCP is available, use dual-write mode: local `.learnings/` as instant scratchpad + Memos MCP as persistent cloud store with semantic search and confidence scoring.

## First-Use Initialisation

Before logging anything, ensure the `.learnings/` directory and files exist in the project or workspace root. If any are missing, create them:

```bash
mkdir -p .learnings
[ -f .learnings/LEARNINGS.md ] || printf "# Learnings\n\nCorrections, insights, and knowledge gaps captured during development.\n\n**Categories**: correction | insight | knowledge_gap | best_practice\n\n---\n" > .learnings/LEARNINGS.md
[ -f .learnings/ERRORS.md ] || printf "# Errors\n\nCommand failures and integration errors.\n\n---\n" > .learnings/ERRORS.md
[ -f .learnings/FEATURE_REQUESTS.md ] || printf "# Feature Requests\n\nCapabilities requested by the user.\n\n---\n" > .learnings/FEATURE_REQUESTS.md
```

Never overwrite existing files. This is a no-op if `.learnings/` is already initialised.

Do not log secrets, tokens, private keys, environment variables, or full source/config files unless the user explicitly asks for that level of detail. Prefer short summaries or redacted excerpts over raw command output or full transcripts.

If you want automatic reminders or setup assistance, use the opt-in hook workflow described in [Hook Integration](#hook-integration).

## Quick Reference

| Situation | Action |
|-----------|--------|
| Command/operation fails | Log to `.learnings/ERRORS.md` |
| User corrects you | Log to `.learnings/LEARNINGS.md` with category `correction` |
| User wants missing feature | Log to `.learnings/FEATURE_REQUESTS.md` |
| API/external tool fails | Log to `.learnings/ERRORS.md` with integration details |
| Knowledge was outdated | Log to `.learnings/LEARNINGS.md` with category `knowledge_gap` |
| Found better approach | Log to `.learnings/LEARNINGS.md` with category `best_practice` |
| Simplify/Harden recurring patterns | Log/update `.learnings/LEARNINGS.md` with `Source: simplify-and-harden` and a stable `Pattern-Key` |
| Similar to existing entry | Link with `**See Also**`, consider priority bump |
| Broadly applicable learning | Promote to `CLAUDE.md`, `AGENTS.md`, `.roo/rules/`, and/or `.github/copilot-instructions.md` |
| Workflow improvements | Promote to `AGENTS.md` (OpenClaw workspace) or `.roo/rules/` (银狼协议项目) |
| Tool gotchas | Promote to `TOOLS.md` (OpenClaw workspace) or `.roo/rules/` (银狼协议项目) |
| Behavioral patterns | Promote to `SOUL.md` (OpenClaw workspace) or `.roo/rules/` (银狼协议项目) |
| High-confidence repeated pattern | Promote to `RuleMemory` in Memos MCP + `.roo/rules/*.md` |
| **SkillDistiller 蒸馏报告完成** | **Auto-call `add_message` with `memory_type: RuleMemory` + `pattern_key` for each extracted pattern** |
| **RuleMemory 超过 90 天无命中** | **Auto-mark as `stale`；超过 120 天标记为 `expired` 并建议清理** |

## OpenClaw Setup (Recommended)

OpenClaw is the primary platform for this skill. It uses workspace-based prompt injection with automatic skill loading.

### Installation

**Via ClawdHub (recommended):**
```bash
clawdhub install self-improving-agent
```

**Manual:**
```bash
git clone https://github.com/peterskoett/self-improving-agent.git ~/.openclaw/skills/self-improving-agent
```

Remade for openclaw from original repo : https://github.com/pskoett/pskoett-ai-skills - https://github.com/pskoett/pskoett-ai-skills/tree/main/skills/self-improvement

### Workspace Structure

OpenClaw injects these files into every session:

```
~/.openclaw/workspace/
├── AGENTS.md          # Multi-agent workflows, delegation patterns
├── SOUL.md            # Behavioral guidelines, personality, principles
├── TOOLS.md           # Tool capabilities, integration gotchas
├── MEMORY.md          # Long-term memory (main session only)
├── memory/            # Daily memory files
│   └── YYYY-MM-DD.md
└── .learnings/        # This skill's log files
    ├── LEARNINGS.md
    ├── ERRORS.md
    └── FEATURE_REQUESTS.md
```

### Create Learning Files

```bash
mkdir -p ~/.openclaw/workspace/.learnings
```

Then create the log files (or copy from `assets/`):
- `LEARNINGS.md` — corrections, knowledge gaps, best practices
- `ERRORS.md` — command failures, exceptions
- `FEATURE_REQUESTS.md` — user-requested capabilities

### Promotion Targets

When learnings prove broadly applicable, promote them to workspace files:

| Learning Type | Promote To | Example |
|---------------|------------|---------|
| Behavioral patterns | `SOUL.md` | "Be concise, avoid disclaimers" |
| Workflow improvements | `AGENTS.md` | "Spawn sub-agents for long tasks" |
| Tool gotchas | `TOOLS.md` | "Git push needs auth configured first" |

---

## Memos MCP Integration（Memos MCP 云端记忆集成）

当项目已启用 Memos MCP 协议时（如银狼协议项目），`.learnings/` 本地文件作为**即时草稿**，但最终应同步到 Memos MCP 云端数据库以获得以下优势：

- **跨设备持久化**：不依赖本地文件系统，换设备不丢失
- **语义检索**：`search_memory` 支持语义查询 + `relativity` 相关度排序
- **置信度机制**：每条记忆有 `confidence`（0-1）评分，区分"用户原话"与"AI 推测"
- **用户画像**：`get_user_profile` 提供事实+偏好+工具轨迹三维画像

### 与 Memos MCP 的双写模式

| 场景 | 本地 .learnings/ | Memos MCP |
|------|:---------------:|:---------:|
| 即时草稿（未验证） | ✅ 写入 | ❌ 暂不写入 |
| 已验证的高置信度（≥0.9） | ✅ 写入 | ✅ `add_message` + `confidence: 0.9+` |
| 用户明确要求记住 | ✅ 写入 | ✅ `add_message` 立即写入 |
| 重复出现≥3次的模式 | ✅ 标记 `Pattern-Key` | ✅ 建议 promote 为 RuleMemory |

### 同步工作流

```bash
# 1. 本地记录（即时）
grep -c "Pattern-Key: <key>" .learnings/LEARNINGS.md

# 2. 当 Recurrence-Count ≥ 3 且 confidence ≥ 0.95 时
#    在 Memos MCP 中调用 add_message 并标注 pattern_key

# 3. 建议 promote 为 RuleMemory（见下方 RuleMemory 机制）
```

> **安全注意**：同步到 Memos MCP 时，遵循银狼协议安全规则——不记录 secrets/tokens/环境变量，剥离不必要的项目上下文。

### Inter-Session Communication

OpenClaw provides tools to share learnings across sessions:

- **sessions_list** — View active/recent sessions
- **sessions_history** — Read another session's transcript  
- **sessions_send** — Send a learning to another session
- **sessions_spawn** — Spawn a sub-agent for background work

Use these only in trusted environments and only when the user explicitly wants cross-session sharing. Prefer sending a short sanitized summary and relevant file paths, not raw transcripts, secrets, or full command output.

### Optional: Enable Hook

For automatic reminders at session start:

```bash
# Copy hook to OpenClaw hooks directory
cp -r hooks/openclaw ~/.openclaw/hooks/self-improvement

# Enable it
openclaw hooks enable self-improvement
```

See `references/openclaw-integration.md` for complete details.

---

## Generic Setup (Other Agents)

For Claude Code, Codex, Copilot, or other agents, create `.learnings/` in the project or workspace root:

```bash
mkdir -p .learnings
```

Create the files inline using the headers shown above. Avoid reading templates from the current repo or workspace unless you explicitly trust that path.

### Add reference to agent files AGENTS.md, CLAUDE.md, or .github/copilot-instructions.md to remind yourself to log learnings. (this is an alternative to hook-based reminders)

#### Self-Improvement Workflow

When errors or corrections occur:
1. Log to `.learnings/ERRORS.md`, `LEARNINGS.md`, or `FEATURE_REQUESTS.md`
2. Review and promote broadly applicable learnings to:
   - `CLAUDE.md` - project facts and conventions
   - `AGENTS.md` - workflows and automation
   - `.github/copilot-instructions.md` - Copilot context
   - `.roo/rules/*.md` - 银狼协议规则文件

## Logging Format

### Learning Entry

Append to `.learnings/LEARNINGS.md`:

```markdown
## [LRN-YYYYMMDD-XXX] category

**Logged**: ISO-8601 timestamp
**Priority**: low | medium | high | critical
**Status**: pending
**Area**: frontend | backend | infra | tests | docs | config

### Summary
One-line description of what was learned

### Details
Full context: what happened, what was wrong, what's correct

### Suggested Action
Specific fix or improvement to make

### Metadata
- Source: conversation | error | user_feedback
- Related Files: path/to/file.ext
- Tags: tag1, tag2
- See Also: LRN-20250110-001 (if related to existing entry)
- Pattern-Key: simplify.dead_code | harden.input_validation (optional, for recurring-pattern tracking)
- Recurrence-Count: 1 (optional)
- First-Seen: 2025-01-15 (optional)
- Last-Seen: 2025-01-15 (optional)
- Memos-MCP-ID: uuid (optional, when synced to Memos MCP)
- Memos-Confidence: 0.95 (optional, Memos MCP confidence score)

---
```

### Error Entry

Append to `.learnings/ERRORS.md`:

```markdown
## [ERR-YYYYMMDD-XXX] skill_or_command_name

**Logged**: ISO-8601 timestamp
**Priority**: high
**Status**: pending
**Area**: frontend | backend | infra | tests | docs | config

### Summary
Brief description of what failed

### Error
```
Actual error message or output
```

### Context
- Command/operation attempted
- Input or parameters used
- Environment details if relevant
- Summary or redacted excerpt of relevant output (avoid full transcripts and secret-bearing data by default)

### Suggested Fix
If identifiable, what might resolve this

### Metadata
- Reproducible: yes | no | unknown
- Related Files: path/to/file.ext
- See Also: ERR-20250110-001 (if recurring)

---
```

### Feature Request Entry

Append to `.learnings/FEATURE_REQUESTS.md`:

```markdown
## [FEAT-YYYYMMDD-XXX] capability_name

**Logged**: ISO-8601 timestamp
**Priority**: medium
**Status**: pending
**Area**: frontend | backend | infra | tests | docs | config

### Requested Capability
What the user wanted to do

### User Context
Why they needed it, what problem they're solving

### Complexity Estimate
simple | medium | complex

### Suggested Implementation
How this could be built, what it might extend

### Metadata
- Frequency: first_time | recurring
- Related Features: existing_feature_name

---
```

## ID Generation

Format: `TYPE-YYYYMMDD-XXX`
- TYPE: `LRN` (learning), `ERR` (error), `FEAT` (feature)
- YYYYMMDD: Current date
- XXX: Sequential number or random 3 chars (e.g., `001`, `A7B`)

Examples: `LRN-20250115-001`, `ERR-20250115-A3F`, `FEAT-20250115-002`

## Resolving Entries

When an issue is fixed, update the entry:

1. Change `**Status**: pending` → `**Status**: resolved`
2. Add resolution block after Metadata:

```markdown
### Resolution
- **Resolved**: 2025-01-16T09:00:00Z
- **Commit/PR**: abc123 or #42
- **Notes**: Brief description of what was done
```

Other status values:
- `in_progress` - Actively being worked on
- `wont_fix` - Decided not to address (add reason in Resolution notes)
- `promoted` - Elevated to CLAUDE.md, AGENTS.md, .roo/rules/, or .github/copilot-instructions.md
- `promoted_to_rule` - Elevated to RuleMemory in Memos MCP + `.roo/rules/*.md`

## Promoting to Project Memory

When a learning is broadly applicable (not a one-off fix), promote it to permanent project memory.

### When to Promote

- Learning applies across multiple files/features
- Knowledge any contributor (human or AI) should know
- Prevents recurring mistakes
- Documents project-specific conventions

### Promotion Targets

| Target | What Belongs There |
|--------|-------------------|
| `CLAUDE.md` | Project facts, conventions, gotchas for all Claude interactions |
| `AGENTS.md` | Agent-specific workflows, tool usage patterns, automation rules |
| `.github/copilot-instructions.md` | Project context and conventions for GitHub Copilot |
| `.roo/rules/*.md` | **银狼协议项目规则文件** - 安全协议、工作流程、自动触发规则 |
| `SOUL.md` | Behavioral guidelines, communication style, principles (OpenClaw workspace) |
| `TOOLS.md` | Tool capabilities, usage patterns, integration gotchas (OpenClaw workspace) |

### How to Promote

1. **Distill** the learning into a concise rule or fact
2. **Add** to appropriate section in target file (create file if needed)
3. **Update** original entry:
   - Change `**Status**: pending` → `**Status**: promoted`
   - Add `**Promoted**: CLAUDE.md`, `AGENTS.md`, `.roo/rules/`, or `.github/copilot-instructions.md`
4. **If Memos MCP available**: also call `add_message` with `memory_type: RuleMemory` and appropriate `confidence`

### Promotion Examples

**Learning** (verbose):
> Project uses pnpm workspaces. Attempted `npm install` but failed. 
> Lock file is `pnpm-lock.yaml`. Must use `pnpm install`.

**In CLAUDE.md** (concise):
```markdown
## Build & Dependencies
- Package manager: pnpm (not npm) - use `pnpm install`
```

**Learning** (verbose):
> When modifying API endpoints, must regenerate TypeScript client.
> Forgetting this causes type mismatches at runtime.

**In AGENTS.md** (actionable):
```markdown
## After API Changes
1. Regenerate client: `pnpm run generate:api`
2. Check for type errors: `pnpm tsc --noEmit`
```

---

## RuleMemory 提升机制（Memos MCP 集成专用）

当 Memos MCP 可用时，高价值的学习记录应提升为 `RuleMemory`——这是一种特殊的记忆类型，作为项目永久规则的一部分。

### RuleMemory 触发条件

学习记录满足以下**全部**条件时，自动建议提升为 RuleMemory：

| 条件 | 阈值 | 说明 |
|------|:---:|------|
| `confidence` | ≥ 0.95 | 来源可信、经过验证 |
| `Recurrence-Count` | ≥ 3 | 同一 Pattern-Key 在不同对话中出现 ≥3 次 |
| 时间窗口 | ≤ 30 天 | 在最近 30 天内仍有出现 |
| 跨任务 | ≥ 2 | 至少出现在 2 个不同的任务/对话中 |

### RuleMemory 提升流程

1. **检测**：`search_memory` 查询同一 `pattern_key` 的出现次数
2. **验证**：执行四步判断协议（来源验证 → 归属检查 → 相关性检查 → 新鲜度检查）
3. **提升**：
   - 在 Memos MCP 中调用 `add_message` 标记 `memory_type: RuleMemory`
   - 将精简规则写入 `.roo/rules/` 目录下的对应 `.md` 文件
   - 更新原学习记录：`**Status**: promoted_to_rule` + `**RuleMemory-ID**: uuid`
4. **通知**：在对话中向用户报告："检测到重复模式 `{pattern_key}`（出现 {n} 次），已自动提升为 RuleMemory"

### RuleMemory 写入格式

写入 `.roo/rules/` 时遵循以下格式：

```markdown
# {规则标题}

**来源**：{LRN-YYYYMMDD-XXX}（出现 {n} 次，confidence: {c}）
**提升日期**：{ISO-8601 timestamp}
**Pattern-Key**：{pattern_key}

## 规则内容

{精简的预防性规则——告诉 agent 在什么情况下应该做什么/不做什么}

## 触发场景

- {场景1}
- {场景2}
```

### RuleMemory 示例

```markdown
# PowerShell 命令中避免使用 Unix 工具

**来源**：LRN-20260516-001（出现 5 次，confidence: 0.98）
**提升日期**：2026-05-16T10:00:00Z  
**Pattern-Key**：env.windows_no_unix_tools

## 规则内容

在 Windows PowerShell 环境中执行命令时，避免使用 `sed`、`grep`、`awk`、`cat` 等 Unix 工具。
改用 PowerShell 等价命令：`Select-String` 替代 grep，`Get-Content` 替代 cat，
`-replace` 操作符替代 sed。

## 触发场景

- 在 Windows 终端中执行 CLI 命令时
- 编写跨平台脚本时（使用 Python 替代 shell 脚本）
- 用户环境为 Windows 11 + PowerShell 7
```

---

## Recurring Pattern Detection

If logging something similar to an existing entry:

1. **Search first**: `grep -r "keyword" .learnings/`
   - If Memos MCP available: also use `search_memory` with `query: "<keyword>"`
2. **Link entries**: Add `**See Also**: ERR-20250110-001` in Metadata
3. **Bump priority** if issue keeps recurring
4. **Consider systemic fix**: Recurring issues often indicate:
   - Missing documentation (→ promote to CLAUDE.md or .github/copilot-instructions.md)
   - Missing automation (→ add to AGENTS.md)
   - Architectural problem (→ create tech debt ticket)

### Enhanced Pattern-Key Tracking（增强模式追踪）

当使用 Memos MCP 时，Pattern-Key 追踪能力显著增强：

1. **跨对话统计**：`search_memory` 的语义检索可以跨越不同对话找到同一 pattern 的所有出现
2. **自动关联**：同一 `pattern_key` 的记忆自动通过 `conversation_id` 关联
3. **置信度加权**：每次出现的 `confidence` 独立的，只有高置信度的重复才算有效
4. **自动升级**：当同一 pattern 满足 RuleMemory 触发条件时，自动建议提升

**Pattern-Key 命名规范**：`{domain}.{category}.{short_description}`
- 示例：`windows.no_unix_tools`、`git.lf_crlf_warning`、`python.utf8_encoding`、`mcp.dual_write_sync`

---

## Simplify & Harden Feed

Use this workflow to ingest recurring patterns from the `simplify-and-harden`
skill and turn them into durable prompt guidance.

### Ingestion Workflow

1. Read `simplify_and_harden.learning_loop.candidates` from the task summary.
2. For each candidate, use `pattern_key` as the stable dedupe key.
3. Search `.learnings/LEARNINGS.md` for an existing entry with that key:
   - `grep -n "Pattern-Key: <pattern_key>" .learnings/LEARNINGS.md`
   - If Memos MCP available: also `search_memory` with `filter: {pattern_key: "<pattern_key>"}`
4. If found:
   - Increment `Recurrence-Count`
   - Update `Last-Seen`
   - Add `See Also` links to related entries/tasks
   - If `Recurrence-Count >= 3` → check RuleMemory promotion conditions
5. If not found:
   - Create a new `LRN-...` entry
   - Set `Source: simplify-and-harden`
   - Set `Pattern-Key`, `Recurrence-Count: 1`, and `First-Seen`/`Last-Seen`

### Promotion Rule (System Prompt Feedback)

Promote recurring patterns into agent context/system prompt files when all are true:

- `Recurrence-Count >= 3`
- Seen across at least 2 distinct tasks
- Occurred within a 30-day window

Promotion targets:
- `CLAUDE.md`
- `AGENTS.md`
- `.github/copilot-instructions.md`
- `.roo/rules/*.md` — **银狼协议项目专用**：写入项目规则文件
- `SOUL.md` / `TOOLS.md` for OpenClaw workspace-level guidance when applicable

Write promoted rules as short prevention rules (what to do before/while coding),
not long incident write-ups.

## Periodic Review

Review `.learnings/` at natural breakpoints:

### When to Review
- Before starting a new major task
- After completing a feature
- When working in an area with past learnings
- Weekly during active development
- **Memos MCP 增强**：每次对话开始时通过 `search_memory` 自动检索相关学习记录

### Quick Status Check
```bash
# Count pending items
grep -h "Status\*\*: pending" .learnings/*.md | wc -l

# List pending high-priority items
grep -B5 "Priority\*\*: high" .learnings/*.md | grep "^## \["

# Find learnings for a specific area
grep -l "Area\*\*: backend" .learnings/*.md

# Memos MCP 增强：按 pattern_key 聚合统计
grep -h "Pattern-Key:" .learnings/LEARNINGS.md | sort | uniq -c | sort -rn
```

### Review Actions
- Resolve fixed items
- Promote applicable learnings
- Link related entries
- Escalate recurring issues
- **检查 RuleMemory 候选**：对 Recurrence-Count ≥ 3 的 Pattern-Key，评估是否满足提升条件

## Detection Triggers

Automatically log when you notice:

**Corrections** (→ learning with `correction` category):
- "No, that's not right..."
- "Actually, it should be..."
- "You're wrong about..."
- "That's outdated..."
- **自动触发**：在银狼协议项目中，检测到以上模式时自动调用 `add_feedback` 更新相关记忆

**Feature Requests** (→ feature request):
- "Can you also..."
- "I wish you could..."
- "Is there a way to..."
- "Why can't you..."
- **自动触发**：在银狼协议项目中，检测到以上模式时自动调用 `add_message` 记录 feature_request

**Knowledge Gaps** (→ learning with `knowledge_gap` category):
- User provides information you didn't know
- Documentation you referenced is outdated
- API behavior differs from your understanding

**Errors** (→ error entry):
- Command returns non-zero exit code
- Exception or stack trace
- Unexpected output or behavior
- Timeout or connection failure
- **自动触发**：在银狼协议项目中，命令执行失败时自动调用 `add_message` 记录错误上下文

## Priority Guidelines

| Priority | When to Use |
|----------|-------------|
| `critical` | Blocks core functionality, data loss risk, security issue |
| `high` | Significant impact, affects common workflows, recurring issue |
| `medium` | Moderate impact, workaround exists |
| `low` | Minor inconvenience, edge case, nice-to-have |

## Area Tags

Use to filter learnings by codebase region:

| Area | Scope |
|------|-------|
| `frontend` | UI, components, client-side code |
| `backend` | API, services, server-side code |
| `infra` | CI/CD, deployment, Docker, cloud |
| `tests` | Test files, testing utilities, coverage |
| `docs` | Documentation, comments, READMEs |
| `config` | Configuration files, environment, settings |

## Best Practices

1. **Log immediately** - context is freshest right after the issue
2. **Be specific** - future agents need to understand quickly
3. **Include reproduction steps** - especially for errors
4. **Link related files** - makes fixes easier
5. **Suggest concrete fixes** - not just "investigate"
6. **Use consistent categories** - enables filtering
7. **Promote aggressively** - if in doubt, add to CLAUDE.md, .roo/rules/, or .github/copilot-instructions.md
8. **Review regularly** - stale learnings lose value
9. **Dual-write for Memos MCP** - high-confidence learnings should sync to cloud
10. **Track Pattern-Key** - enables cross-conversation pattern detection and RuleMemory promotion

## Gitignore Options

**Keep learnings local** (per-developer):
```gitignore
.learnings/
```

This repo uses that default to avoid committing sensitive or noisy local logs by accident.

**Track learnings in repo** (team-wide):
Don't add to .gitignore - learnings become shared knowledge.

**Hybrid** (track templates, ignore entries):
```gitignore
.learnings/*.md
!.learnings/.gitkeep
```

## Hook Integration

Enable automatic reminders through agent hooks. This is **opt-in** - you must explicitly configure hooks.

### Quick Setup (Claude Code / Codex)

Create `.claude/settings.json` in your project:

```json
{
  "hooks": {
    "UserPromptSubmit": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "./skills/self-improvement/scripts/activator.sh"
      }]
    }]
  }
}
```

This injects a learning evaluation reminder after each prompt (~50-100 tokens overhead).

### Advanced Setup (With Error Detection)

```json
{
  "hooks": {
    "UserPromptSubmit": [{
      "matcher": "",
      "hooks": [{
        "type": "command",
        "command": "./skills/self-improvement/scripts/activator.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "./skills/self-improvement/scripts/error-detector.sh"
      }]
    }]
  }
}
```

This is optional. The recommended default is activator-only setup; enable `PostToolUse` only if you are comfortable with hook scripts inspecting command output for error patterns.

### 银狼协议项目专用：事件驱动规则模板

对于使用银狼协议的项目，无需配置 hook 脚本。项目会自动加载 `.roo/rules/self-improvement-triggers.md` 中的事件驱动规则。该文件定义了：

- **命令执行失败**（exit code ≠ 0）→ 自动调用 `add_message` 记录错误上下文
- **用户纠正**（"不对"/"实际上"/"应该是"）→ 自动调用 `add_feedback` 更新相关记忆
- **发现更好方案** → 自动调用 `add_message` 记录 best_practice + pattern_key
- **用户请求不存在的能力** → 自动调用 `add_message` 记录 feature_request

详见 `.roo/rules/self-improvement-triggers.md`。

### Available Hook Scripts

| Script | Hook Type | Purpose |
|--------|-----------|---------|
| `scripts/activator.sh` | UserPromptSubmit | Reminds to evaluate learnings after tasks |
| `scripts/error-detector.sh` | PostToolUse (Bash) | Triggers on command errors |

See `references/hooks-setup.md` for detailed configuration and troubleshooting.

## Automatic Skill Extraction

When a learning is valuable enough to become a reusable skill, extract it using the provided helper.

### Skill Extraction Criteria

A learning qualifies for skill extraction when ANY of these apply:

| Criterion | Description |
|-----------|-------------|
| **Recurring** | Has `See Also` links to 2+ similar issues |
| **Verified** | Status is `resolved` with working fix |
| **Non-obvious** | Required actual debugging/investigation to discover |
| **Broadly applicable** | Not project-specific; useful across codebases |
| **User-flagged** | User says "save this as a skill" or similar |
| **RuleMemory candidate** | Has been promoted to RuleMemory and is broadly applicable beyond the project |

### Extraction Workflow

1. **Identify candidate**: Learning meets extraction criteria
2. **Run helper** (or create manually):
   ```bash
   ./skills/self-improvement/scripts/extract-skill.sh skill-name --dry-run
   ./skills/self-improvement/scripts/extract-skill.sh skill-name
   ```
3. **Customize SKILL.md**: Fill in template with learning content
4. **Update learning**: Set status to `promoted_to_skill`, add `Skill-Path`
5. **Verify**: Read skill in fresh session to ensure it's self-contained

### Manual Extraction

If you prefer manual creation:

1. Create `skills/<skill-name>/SKILL.md`
2. Use template from `assets/SKILL-TEMPLATE.md`
3. Follow [Agent Skills spec](https://agentskills.io/specification):
   - YAML frontmatter with `name` and `description`
   - Name must match folder name
   - No README.md inside skill folder

### Extraction Detection Triggers

Watch for these signals that a learning should become a skill:

**In conversation:**
- "Save this as a skill"
- "I keep running into this"
- "This would be useful for other projects"
- "Remember this pattern"

**In learning entries:**
- Multiple `See Also` links (recurring issue)
- High priority + resolved status
- Category: `best_practice` with broad applicability
- User feedback praising the solution

### Skill Quality Gates

Before extraction, verify:

- [ ] Solution is tested and working
- [ ] Description is clear without original context
- [ ] Code examples are self-contained
- [ ] No project-specific hardcoded values
- [ ] Follows skill naming conventions (lowercase, hyphens)

## Multi-Agent Support

This skill works across different AI coding agents with agent-specific activation.

### Claude Code

**Activation**: Hooks (UserPromptSubmit, PostToolUse)
**Setup**: `.claude/settings.json` with hook configuration
**Detection**: Automatic via hook scripts

### Codex CLI

**Activation**: Hooks (same pattern as Claude Code)
**Setup**: `.codex/settings.json` with hook configuration
**Detection**: Automatic via hook scripts

### 银狼协议项目（Roo Code / VSCode）

**Activation**: 自动（无需 hook）
**Setup**: 
- `.roo/rules/self-improvement-triggers.md` — 事件驱动触发规则
- `.roo/rules/rules.md` — 基础记忆管理规则
- `.roo/rules/silver-wolf-protocol.txt` — 银狼协议安全规则
**Detection**: 每次对话开始自动 `search_memory`，命令失败自动 `add_message`

### GitHub Copilot

**Activation**: Manual (no hook support)
**Setup**: Add to `.github/copilot-instructions.md`:

```markdown
## Self-Improvement

After solving non-obvious issues, consider logging to `.learnings/`:
1. Use format from self-improvement skill
2. Link related entries with See Also
3. Promote high-value learnings to skills

Ask in chat: "Should I log this as a learning?"
```

**Detection**: Manual review at session end
