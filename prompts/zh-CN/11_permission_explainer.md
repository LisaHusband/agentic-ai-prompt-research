# 权限解释器系统提示词

> **观察自**：Claude Code 内部架构
>
> 一个轻量的侧查询（side query），在用户批准某个工具/命令之前，解释它做什么、为什么要运行它以及它的风险水平。使用结构化工具输出以保证返回 JSON 响应。

---

## 系统提示词

```
分析 shell 命令并解释它们做什么、你为什么要运行它们以及潜在的风险。
```

## 结构化输出 Schema（工具：`explain_command`）

```json
{
  "explanation": "这条命令做什么（1-2 句话）",
  "reasoning": "你为什么运行这条命令。以 'I' 开头——例如 'I need to check the file contents'",
  "risk": "可能出什么问题，15 个词以内",
  "riskLevel": "LOW | MEDIUM | HIGH"
}
```

### 风险等级定义

| 等级 | 描述 | 示例 |
|-------|-------------|---------|
| `LOW` | 安全的开发工作流 | `ls`、`cat`、`git status` |
| `MEDIUM` | 可恢复的改动 | 文件编辑、`npm install` |
| `HIGH` | 危险/不可逆 | `rm -rf`、`DROP TABLE`、force push |

## 运行时行为

- 使用**主循环模型**（而不是单独的 Haiku 模型）
- 作为**侧查询**与权限提示并发触发
- 提取最近 3 条助手消息（最多 1000 字符）作为“为什么”的背景
- 可由用户配置禁用：`permissionExplainerEnabled: false`
