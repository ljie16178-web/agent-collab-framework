# 自动学习触发规则（Self-Improvement Triggers）

> **来源**：基于 self-improving-agent 技能的改进
> **关联文件**：`skills/self-improving-agent/SKILL.md`、`.roo/rules/rules.md`、`.roo/rules/silver-wolf-protocol.txt`

## 核心原则

- 每次对话开始前自动调用 `search_memory` 检索相关学习记录
- 命令执行失败或用户纠正时自动触发学习记录
- 高价值重复模式经用户确认后可提升为 RuleMemory

---

## 触发规则1：命令执行失败 → 自动记录错误

**触发条件**：CLI 命令返回非零 exit code

**执行动作**：
```
调用 mcp--memos-api-mcp--add_message:
  - conversation_first_message: 当前对话首条消息
  - messages: [{
      role: "assistant",
      content: "[错误报告] 命令: <command> | 退出码: <exit_code> | 错误信息: <error_summary>"
    }]
```

**同时**：在 `.learnings/{PROJECT_NAME}/errors.md` 中追加本地记录（PROJECT_NAME 为当前项目名，从 `.roo/skills/auto_prompt_instance.md` 的 `{{PROJECT_NAME}}` 获取）。格式：
```markdown
## [ERR-YYYYMMDD-XXX] <command_name>

**Logged**: ISO-8601 timestamp
**Priority**: high
**Status**: pending
**Area**: config | infra
**Pattern-Key**: <command.category.description>

### Summary
命令 `<command>` 执行失败（exit code: <code>）

### Error
```
<redacted error output>
```

### Context
- 工作目录: <cwd>
- 环境: Windows 11, PowerShell 7

### Suggested Fix
<从经验中推断的修复方案>
```

**Tags 要求**：所有追加内容必须携带项目 Tags（见下方 Tags 规范）。

---

## 触发规则2：用户纠正 → 自动更新记忆

**触发条件**：检测到以下关键短语之一
- "不对"
- "实际上"
- "应该是"
- "错了"

**执行动作**：
```
调用 mcp--memos-api-mcp--add_feedback:
  - conversation_first_message: 当前对话首条消息
  - feedback_content: "用户纠正: <纠正内容的精简摘要>"
```

**同时**：在 `.learnings/{PROJECT_NAME}/{YYYY-MM}.md` 中追加本地记录，category: `correction`。

**Tags 要求**：追加时在文件顶部确认 Tags 行存在，格式为 `# Tags: [{PROJECT_NAME}] [{CATEGORY}] [{YYYY-MM}]`。所有条目带 Pattern-Key 标注。

---

## 触发规则3：发现更好方案 → 自动记录 best_practice

**触发条件**：在完成任务后发现或采用了比初始方案更好的方法

**执行动作**：
```
调用 mcp--memos-api-mcp--add_message:
  - conversation_first_message: 当前对话首条消息
  - messages: [{
      role: "assistant",
      content: "[最佳实践] pattern_key: <key> | 标题: <summary> | 详情: <details>"
    }]
```

**同时**：在 `.learnings/{PROJECT_NAME}/{YYYY-MM}.md` 中追加本地记录，category: `best_practice`，带 `Pattern-Key`。

**Tags 要求**：追加时携带项目 Tags，格式为 `# Tags: [{PROJECT_NAME}] [{best_practice}] [{YYYY-MM}]`。

---

## 触发规则4：用户请求不存在的能力 → 自动记录 feature_request

**触发条件**：检测到以下关键短语之一
- "能不能..."
- "可以...吗"
- "有没有办法..."
- "为什么不能..."

**执行动作**：
```
调用 mcp--memos-api-mcp--add_message:
  - conversation_first_message: 当前对话首条消息
  - messages: [{
      role: "assistant",
      content: "[功能请求] 能力: <capability> | 用户需求: <user_context>"
    }]
```

---

## 触发规则5：重复模式检测 → 自动建议 RuleMemory 提升

**前置动作（grep 预过滤）**：
```
1. grep -r "Pattern-Key:" .learnings/{PROJECT_NAME}/  ← 提取所有 Pattern-Key
2. 统计每个 Pattern-Key 出现次数
3. 按出现次数降序，取 ≥ 3 次的条目
4. 只把这几条（约5-20条）喂给 LLM 做模式分析
```

**触发条件**：在预过滤结果中发现同一 `Pattern-Key` 出现 ≥ 3 次

**执行动作**：
1. 计算 Recurrence-Count + 平均 confidence
2. 如果满足 RuleMemory 条件（Recurrence-Count ≥ 3, confidence ≥ 0.95, ≤ 30 天窗口, ≥ 2 个不同任务）：
   - 在对话中通知用户："检测到重复模式 `{pattern_key}`（出现 {n} 次），建议提升为 RuleMemory"
   - 用户确认后，写入 `.roo/rules/{pattern_key}.md`
3. 如果用户未确认，标记为 `**Priority**: high` 待下次审核

---

## 触发规则6：高置信度学习 → 自动同步到 Memos MCP

**触发条件**：`.learnings/` 中的记录被标记为 `**Status**: resolved` 且 confidence ≥ 0.9

**执行动作**：
```
调用 mcp--memos-api-mcp--add_message:
  - 同步到云端，标注 Memos-MCP-ID 和 Memos-Confidence
```

**回写**：在本地 `.learnings/` 记录中添加：
```markdown
- Memos-MCP-ID: <uuid>
- Memos-Confidence: 0.95
```

---

## 触发规则7：任务完成 → 自动触发 SkillRefiner

**触发条件**：
- `TaskUpdate status=completed`（任意任务完成时）
- 或用户明确说"任务完成"/"搞定了"/"done"

**执行动作**：
1. 读取本次任务的完整对话上下文
2. 分析本次任务中用到了哪些 skill（从对话中识别 skill 调用信号）
3. 按 SkillRefiner 的评估标准判断是否需要蒸馏
4. 如果需要：
   - 生成项目专用版本内容
   - 在对话中通知用户
   - 等待确认后写入 `.roo/skills/<project>-<skill-name>.md`

**SkillRefiner 工作流**（详细执行标准见 `skills/skill-refiner/SKILL.md`）：

```
任务完成
  ↓
Step 1：分析用到了哪些 skill + 使用频率
  ↓
Step 2：检查是否已有项目专用版
  ↓
Step 3：评估提炼价值（频率×适配度）
  ↓
Step 4：生成项目专用版本（如果价值高）
  ↓
Step 5：通知用户确认
  ↓
Step 6：写入 .roo/skills/<project>-<skill-name>.md
```

**提炼约束**：
- 所有结果都是**新生成文件**，不修改原始通用 skill
- 通用 skill 路径：`skills/` 或原始发布路径
- 项目专用版路径：`.roo/skills/<project>-<skill-name>.md`

**反向上化**：如果某 skill 在项目中被频繁纠正，记录到 `.learnings/` 并在 `.roo/rules/` 中写入否定规则。

---

## 文件结构要求

触发规则自动维护以下文件结构：

```
项目根目录/
├── .learnings/                          # 本地即时草稿（按项目/日期分文件）
│   └── {project-name}/
│       └── {YYYY-MM}.md                # 例：lamda-power-trading/2026-05.md
├── .roo/
│   ├── rules/
│   │   ├── rules.md                     # 基础规则（含轮次累积触发层）
│   │   ├── .turn_count.json            # 轮次计数器（运行时状态，gitignore）
│   │   └── self-improvement-triggers.md # 本文件（含规则1-7）
│   └── skills/                          # Roo 内置 skill（项目专用绑定实例）
│       ├── auto_prompt_instance.md      # 三Agent角色绑定
│       ├── skilldistiller_instance.md  # 技能蒸馏绑定
│       └── Anti_AI_Flavor_Engine_Pro.md # 去AI味引擎绑定
├── skills/                               # 通用技能库（跨项目复用）
│   ├── skill-refiner/                   # 提炼工具
│   │   └── SKILL.md
│   └── ...（其他通用skills）
└── prompts/                             # 项目提示词备份
    ├── rules.md
    └── silver-wolf-protocol.txt
```

---

## LEARNINGS 管理规范（Tags + grep 预过滤）

### Tags 规范

每个 `.learnings/` 文件顶部必须包含 Tags 行，不得缺失：

```markdown
# Tags: [{PROJECT_NAME}] [{CATEGORY}] [{YYYY-MM}]
```

| 维度 | 示例 | 说明 |
|------|------|------|
| PROJECT_NAME | `lamda-power-trading` | 项目名，从 auto_prompt_instance.md 获取 |
| CATEGORY | `三agent协作` `错误记录` `最佳实践` `功能请求` | 当前条目类型 |
| YYYY-MM | `2026-05` | 记录所属月份 |

**规则**：所有追加操作必须携带对应 Tags；Tags 行缺失时自动补充。

### grep 预过滤流程

**适用场景**：Rule 5（重复模式检测）、SkillDistiller 启动时、用户请求"检查错误模式"时。

**执行步骤**：
```
1. 获取当前 PROJECT_NAME（从 auto_prompt_instance.md 读取）
2. 确定搜索范围：.learnings/{PROJECT_NAME}/ 目录下所有 .md 文件
3. 执行 grep 预过滤：
   - 按 PROJECT_NAME Tags 过滤：grep -r "# Tags:.*\[{PROJECT_NAME}\]" .learnings/
   - 按条目类型过滤（如只查 errors）：grep -r "\[ERR-" .learnings/{PROJECT_NAME}/
   - 按 Pattern-Key 统计：提取所有 Pattern-Key，统计出现次数
4. 将过滤后的小规模结果（约5-20条）喂给 LLM 做模式分析
5. 大文件（>100条）先按日期分段再处理
```

**grep 命令参考**：
```bash
# 查找当前项目所有错误记录
grep -r "# Tags:.*\[lamda-power-trading\].*\[错误记录\]" .learnings/lamda-power-trading/

# 统计 Pattern-Key 出现次数
grep -rh "Pattern-Key:" .learnings/lamda-power-trading/*.md | sort | uniq -c | sort -rn

# 查找 pending 状态错误
grep -r "Status: pending" .learnings/lamda-power-trading/errors.md
```

### 长期膨胀处理

- 单文件超过 100 条时，按月份自动拆分
- ERRORS.md 超过 50 条时，旧的 resolved 条目移入 `.learnings/{PROJECT_NAME}/archive/`
- FEATURE_REQUESTS.md 超过 30 条时，同上处理

---

## 更新日期

2026-05-22
