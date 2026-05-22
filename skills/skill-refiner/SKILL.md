---
name: skill-refiner
description: "从通用skill自动蒸馏项目专用版本。触发条件：任务完成时自动激活（通过 self-improvement-triggers 规则7）。核心功能：分析本次对话用到的skill → 检查是否有项目专用版 → 生成优化后的项目专用版本（不改原始通用版）。"
---

# SkillRefiner：通用skill → 项目专用skill 蒸馏器

## 核心原则

**不修改原始通用skill。** 所有提炼结果都是**新生成**一个项目专用版本，写入 `.roo/skills/<project>-<skill-name>.md`，原始通用版保留不动。

---

## 触发时机

**规则7**（在 `self-improvement-triggers.md` 中定义）：
```
触发条件：TaskUpdate status=completed 或用户明确说"任务完成"
执行动作：自动调用 SkillRefiner
```

---

## 工作流程

```
任务完成
  ↓
Step 1：分析本次对话用到了哪些skill
  ↓
Step 2：检查该项目是否已有该skill的专用版本
  ↓
Step 3：评估提炼价值（命中率 × 项目适配度）
  ↓
Step 4：生成项目专用版本（如果价值高）
  ↓
Step 5：通知用户并等待确认
  ↓
Step 6：写入 .roo/skills/<project>-<skill-name>.md
```

---

## Step 1：分析skill使用情况

**输入**：当前对话的完整上下文（用户本次任务描述 + AI响应历史）

**方法**：
1. 从对话中识别"调用了哪个skill"的信号：
   - 用户说"用xxx skill" / "触发xxx"
   - 对话中出现了skill名称（如 `/去AI味`、`/pdf`、`lambda_excel_v1`）
   - AI明确引用了某个skill的指导内容
2. 记录该skill在本次任务中的：
   - **使用次数**（hit_count）
   - **用户反馈**（纠正次数、满意度）
   - **解决的问题类型**

**输出**：
```markdown
## 本次任务 skill 使用记录

| Skill名称 | 使用次数 | 用户纠正 | 评价 |
|-----------|---------|---------|------|
| 去AI味.skill | 3 | 0 | 良好 |
| lambda_excel_v1 | 2 | 1 | 需调整 |
```

---

## Step 2：检查项目专用版是否存在

**检查路径**：
```bash
.roo/skills/<project>-<skill-name>.md    # 例如：lamda-去AI味.md
skills/<project>-<skill-name>.md          # 备选路径
```

**判断逻辑**：
- 如果**项目专用版已存在** → 标记为 `exists`，不重复生成
- 如果**不存在** → 进入 Step 3

---

## Step 3：评估提炼价值

| 维度 | 评估标准 | 权重 |
|------|---------|------|
| **使用频率** | 该skill在本次任务中使用 ≥ 2 次 | 高 |
| **用户反馈** | 无纠正或正面评价 | 高 |
| **项目适配度** | 通用版有明显可改进空间（如：参数调整、项目特定路径、术语替换） | 中 |
| **可复用性** | 该skill可能用于未来其他类似任务 | 中 |

**提炼价值阈值**：
- 使用频率 ≥ 2 + 无纠正 → **建议提炼**
- 使用频率 ≥ 2 + 有纠正 → **待观察，下次有纠正再提炼**
- 使用频率 = 1 → 一般不提炼，除非项目适配度极高

---

## Step 4：生成项目专用版本

**提炼内容**（从通用版到项目专用版）：

| 维度 | 通用版 | 项目专用版 |
|------|--------|----------|
| **路径** | `skills/xxx.skill/` | `.roo/skills/<project>-xxx.md` |
| **描述** | 通用描述 | 加入项目特定背景和参数 |
| **触发条件** | 通用触发 | 项目特有的任务类型 |
| **使用示例** | 通用例子 | 项目相关例子 |
| **参数** | 默认参数 | 项目特定路径/术语 |

**生成原则**：
- 保留通用版的核心逻辑
- 只在描述、示例、参数上做项目定制
- 明确标注"基于 XXX.skill 蒸馏，项目专用版"
- 标注提炼日期和来源

---

## Step 5：通知用户

**提示格式**：
```
[SkillRefiner] 检测到「{skill名称}」在本次任务中使用 {n} 次，
无用户纠正，已生成项目专用版本。

路径：.roo/skills/{project}-{skill-name}.md
核心改动：{1-2句话说明项目适配点}

是否启用？（Y/n）
```

**用户确认后** → 写入文件
**用户拒绝** → 不写入，在本次对话中继续使用通用版

---

## Step 6：写入文件

**文件路径**：
```
.roo/skills/<project>-<skill-name>.md
```

**文件格式**：
```markdown
---
name: {project}-{skill-name}
description: "{项目名}项目专用版。基于 {通用skill} 蒸馏。"
source_skill: {通用skill路径}
distilled_date: {ISO-8601}
distilled_reason: "{提炼原因}"
---

# {项目名}：{skill名称}（项目专用版）

## 与通用版的差异

{说明项目定制的内容}

## 使用场景

{项目特有的触发场景}

## 使用示例

{项目相关的例子}
```

---

## 输出示例

假设本次任务用了`去AI味.skill`3次，效果良好，生成了：

**文件**：`.roo/skills/lamda-去AI味.md`
```markdown
---
name: lamda-去AI味
description: "lamda-power-trading项目专用版，基于去AI味.skill蒸馏。"
source_skill: skills/去AI味.skill
distilled_date: 2026-05-22T14:25:00+08:00
distilled_reason: "本次论文写作任务中使用了3次，用户无纠正，生成项目专用版。"
---

# lamda-power-trading：去AI味（项目专用版）

## 与通用版的差异

- 保留通用版的禁用词表和弹性设计原则
- 追加 lamda-power-trading 项目术语：电力现货市场、火电边际成本、月报交付
- 写作示例改为论文/报告场景

## 使用场景

- 论文写作（摘要、引言、结论）
- 月报撰写（太保资本9省份电力数据分析报告）
- 央企/国企风格报告（去官样文章）

## 使用示例

用户输入AI味段落：
> "随着电力市场改革的不断深入，现货市场价格信号..."

输出改写后：
> "现货市场改革推进后，2024年1-4月河北南网日前价格均值为..."
```

---

## 安全约束

1. **不覆盖原始通用skill** — 所有蒸馏结果都是新文件
2. **不记录敏感信息** — 不把本次任务的私密内容写入skill
3. **用户确认后才写入** — 蒸馏结果只是建议，不自动生效
4. **可追溯** — 每个项目专用版都标注 source_skill 和蒸馏日期

## 反向上化（Negative Distillation）

如果某skill在项目中使用时**频繁被纠正**：
- 在 LEARNINGS.md 中记录：`[distillation_rejected] skill: xxx | 纠正原因: {summary}`
- 不生成项目专用版，而是在 `.roo/rules/` 中写入一条"该项目不使用xxx技能"的规则
