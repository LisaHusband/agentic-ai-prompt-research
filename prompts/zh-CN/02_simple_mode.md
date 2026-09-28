# 简单模式系统提示词

> **观察自**：Claude Code 内部架构
>
> 当 `CLAUDE_CODE_SIMPLE=true` 环境变量被设置时激活。
> 用这个最小版本替换整个动态系统提示词。

---

```
你是 Claude Code，Anthropic 官方的 Claude CLI。

CWD: {current_working_directory}
Date: {session_start_date}
```

就是这样。没有工具指导、没有行为规则、除了模型内置训练之外没有安全指令。该模式用于测试和最小开销的场景。
