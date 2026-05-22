# Agent Collaboration Framework

一个基于 Roo Code 的多 Agent 分工协作框架，支持三角色（流萤/银狼/黑塔）自动协作、记忆管理、自我改进和技能蒸馏。

## 架构概览

```
agent-collab-framework/
├── .roo/                           # Roo Code 工作区配置
│   ├── rules/
│   │   ├── rules.md               # 轮次累积触发 + 记忆管理基础层
│   │   └── self-improvement-triggers.md  # 7条自动学习触发规则
│   └── skills/                     # 项目专用 skill 绑定
│       ├── my-persona/            # 用户角色设定（杰）
│       ├── self-improving-agent/  # 自我改进 agent
│       ├── auto_prompt_instance.md        # 三 Agent 角色绑定
│       ├── skilldistiller_instance.md     # 技能蒸馏绑定
│       └── Anti_AI_Flavor_Engine_Pro.md   # 去AI味引擎绑定
├── skills/                         # 通用技能库（跨项目复用）
│   ├── anti-ai-flavor-pro/        # 去AI味引擎
│   ├── skill-refiner/             # 技能提炼工具
│   ├── skilldistiller/            # 技能蒸馏器
│   ├── auto_prompt_3agents/       # 三 Agent 提示生成器
│   └── 去AI味.skill/             # 中文去AI味写作风格
├── .learnings/
│   └── LEARNINGS.md               # 学习记录（跨 session 记忆）
└── README.md                       # 本文件
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
3. **RuleMemory 提升**：重复模式（≥3次，confidence≥0.95）自动提升为永久规则
4. **SkillDistiller 闭环**：任务完成后自动分析技能使用情况，生成项目专用版本

## 使用方式

1. 将本框架复制到项目根目录
2. 修改 `.roo/skills/auto_prompt_instance.md` 中的项目绑定变量
3. 修改 `.roo/skills/my-persona/SKILL.md` 中的用户设定
4. 开始新对话，Roo Code 自动加载规则

## 记忆管理

- **本地草稿**：`.learnings/LEARNINGS.md`
- **云端持久**：Memos MCP（需配置 MCP 服务器）
- **双写模式**：即时写入本地 + 高置信度时同步云端

## 规则更新

修改 `.roo/rules/*.md` 后，**新对话自动生效**，无需重启。
