# Tags: [{PROJECT_NAME}] [{错误记录}] [{YYYY-MM}]

---

# 错误记录模板

> 项目：{PROJECT_NAME}
> 用途：本文件记录命令执行失败、工具调用错误等，每次错误单独一个条目

---

## 错误条目模板

```markdown
## [ERR-YYYYMMDD-XXX] <command_name>

**Logged**: ISO-8601 timestamp
**Priority**: high | medium | low
**Status**: pending | resolved | closed
**Area**: config | infra | code | docs
**Pattern-Key**: <command.category.description>

### Summary
命令 `<command>` 执行失败（exit code: <code>）

### Error
```
<error output>
```

### Context
- 工作目录: <cwd>
- 环境: <environment>

### Suggested Fix
<从经验中推断的修复方案>
```

---

**使用说明**：
- 每次命令失败时追加条目到本文件
- **Pattern-Key** 必须填写，用于 Rule 5 重复模式检测
- `Status: pending` → 修复后改为 `resolved`
- 超过 50 条 pending 时，旧的 resolved 条目移入 `archive/` 子目录
