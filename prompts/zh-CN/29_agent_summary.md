# Agent 摘要（后台进度）

**观察自**：Claude Code 内部架构
**模型：** 小型快速模型（Haiku）

## 用途

为在 coordinator 模式下运行的子代理生成周期性的后台进度更新。让父 agent 实时了解每个 worker 正在做什么。

## 提示词（从源码分析重建）

```
用 1 个短句概括该 agent 当前正在做什么。使用现在时。
聚焦具体的动作，而不是整体任务。

好：“Reading the authentication middleware to understand token validation”
好：“Running pytest on the user service after fixing the import error”
差：“Working on the task”（太含糊）
差：“The agent is currently in the process of examining...”（太啰嗦）
```

> 注：示例保留英文原文，因为该提示词明确要求使用现在时动词（“Reading”“Running”“Fixing”），其时态演示只对英文有意义。

## 设计约束

- **单个句子**：最多一个句子，使用现在时
- **动作具体**：描述当前动作，而不是整体目标
- **没有元评论**：不要描述 agent 本身，而要描述它正在做什么
- **现在时**：始终使用现在时动词（“Reading”“Running”“Fixing”）

## 集成点

- 在 coordinator 模式期间由周期性定时器运行
- 结果显示在 coordinator 的状态视图中
- 帮助主导 agent 决定何时去查看各个 worker
- 只在子代理正在积极执行工具调用时触发
