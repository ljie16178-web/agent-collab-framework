# Agent Collaboration Framework

一个基于 Roo Code 的多 Agent 分工协作框架，支持三角色（流萤/银狼/黑塔）自动协作、记忆管理、自我改进和技能蒸馏。

## 架构概览

```
agent-collab-framework/
├── .roo/                                # Roo Code 工作区配置
│   ├── rules/
│   │   ├── rules.md                     # 轮次累积触发 + 记忆管理基础层
│   │   ├── .turn_count.json            # 轮次计数器（运行时状态，gitignore）
│   │   └── self-improvement-triggers.md # 7条自动学习触发规则 + LEARNINGS管理规范
│   └── skills/                          # 项目专用 skill 绑定（实例配置）
│       ├── my-persona/                  # 用户角色设定
│       ├── self-improving-agent/        # 自我改进 agent
│       ├── auto_prompt_instance.md      # 三Agent角色绑定（需按项目修改）
│       ├── skilldistiller_instance.md  # 技能蒸馏绑定（需按项目修改）
│       └── Anti_AI_Flavor_Engine_Pro.md # 去AI味引擎绑定
├── skills/                              # 通用技能库（跨项目复用）
│   ├── anti-ai-flavor-pro/             # 去AI味引擎
│   ├── skill-refiner/                  # 技能提炼工具（通用）
│   ├── skilldistiller/                 # 技能蒸馏器（通用）
│   ├── auto_prompt_3agents/            # 三Agent提示生成器
│   └── 去AI味.skill/                   # 中文去AI味写作风格
├── .learnings/                          # 学习记录（按项目/日期分文件，模板为空）
│   └── {project-name}/                  # 例：lamda-power-trading/
│       └── {YYYY-MM}.md               # 例：2026-05.md
└── README.md                            # 本文件
```

## 三 Agent 分工

| 角色 | 名称 | 模式 | 职责 |
|------|------|------|------|
| 总设计师 | 流萤 | architect | 架构设计、任务分解、方案规划 |
| 总代码师 | 银狼 | code | 代码实现、逻辑开发、技术落地 |
| 总审核师 | 黑塔 | ask | 代码审核、逻辑验证、质量把控 |

## 核心机制

1. **轮次累积触发**：每5轮对话自动触发复盘提醒，沉淀经验到 `.learnings/`
2. **自动学习触发**：命令失败/用户纠正/发现更好方案时自动记录
3. **RuleMemory 提升**：重复模式（≥3次，confidence≥0.95）**用户确认后**提升为永久规则
4. **SkillDistiller 闭环**：任务完成后自动分析技能使用情况，生成项目专用版本

## 使用方式

1. 将本框架复制到项目根目录
2. 修改 `.roo/skills/auto_prompt_instance.md` 中的项目绑定变量（PROJECT_NAME 等）
3. 修改 `.roo/skills/my-persona/SKILL.md` 中的用户设定
4. 初始化 `.learnings/{PROJECT_NAME}/` 目录（可从模板复制）
5. 开始新对话，Roo Code 自动加载规则

## 记忆管理（LEARNINGS）

### 文件结构

`.learnings/` 按**项目/日期**分文件，不再使用单一 LEARNINGS.md：

```
.learnings/
└── {project-name}/          # 项目名，从 auto_prompt_instance.md 获取
    ├── {YYYY-MM}.md        # 月度学习记录
    └── errors.md            # 错误记录（可选）
```

### Tags 规范

每个 `.learnings/` 文件顶部必须有 Tags 行，用于 grep 预过滤：

```markdown
# Tags: [{PROJECT_NAME}] [{CATEGORY}] [{YYYY-MM}]
```

| 维度 | 示例 | 说明 |
|------|------|------|
| PROJECT_NAME | `lamda-power-trading` | 项目名 |
| CATEGORY | `三agent协作` `错误记录` `最佳实践` | 条目类型 |
| YYYY-MM | `2026-05` | 记录所属月份 |

### grep 预过滤

**Rule 5（重复模式检测）**和 **SkillDistiller** 启动前，先用 grep 预过滤，减少上下文膨胀：

```bash
# 统计 Pattern-Key 出现次数
grep -rh "Pattern-Key:" .learnings/{PROJECT_NAME}/*.md | sort | uniq -c | sort -rn

# 查找 pending 状态错误
grep -r "Status: pending" .learnings/{PROJECT_NAME}/errors.md

# 按项目 Tags 过滤
grep -r "# Tags:.*\[{PROJECT_NAME}\]" .learnings/{PROJECT_NAME}/
```

过滤后只将 5-20 条相关条目喂给 LLM，避免上下文超载。

### 云端持久

- **Memos MCP**：高置信度规则（confidence ≥ 0.9）同步到云端
- **双写模式**：即时写入本地 `.learnings/` + 满足条件时同步 Memos MCP

## 规则更新

修改 `.roo/rules/*.md` 后，**新对话自动生效**，无需重启。
