# YOLO/Auto 模式分类器系统

> **观察自**：Claude Code 内部架构
>
> 一个 2 阶段的安全分类系统，用于判定工具调用应当被自动批准还是被阻止。这是 Claude Code“自动批准”模式的核心。

---

## 架构概览

YOLO 分类器是 Claude Code **最关乎安全的关键组件**。当用户启用“自动模式”（此前称为“YOLO 模式”）时，该分类器会在每一次工具调用运行之前对其进行评估。

### 工作方式

1. **基础提示词**：从 `yolo-classifier-prompts/auto_mode_system_prompt.txt` 加载（构建时通过 `bun:bundle` 特性开关 `TRANSCRIPT_CLASSIFIER` 打包）
2. **权限模板**：运行时根据用户类型进行替换：
   - **外部用户**：`permissions_external.txt`
   - **Anthropic 员工**（`USER_TYPE=ant`）：`permissions_anthropic.txt`
3. **用户规则**：从 `settings.autoMode` 注入三个可定制小节：
   - `allow`——用户明确允许的动作
   - `soft_deny`——应当被阻止的动作（但用户可以覆盖）
   - `environment`——关于用户环境的背景信息

### 分类器工具 Schema

分类器使用一次强制工具调用（`classify_result`）来获得结构化输出：

```json
{
  "name": "classify_result",
  "description": "报告针对该 agent 动作的安全分类结果",
  "input_schema": {
    "type": "object",
    "properties": {
      "thinking": {
        "type": "string",
        "description": "简要的逐步推理。"
      },
      "shouldBlock": {
        "type": "boolean",
        "description": "该动作应当被阻止（true）还是被允许（false）"
      },
      "reason": {
        "type": "string",
        "description": "对分类决定的简要解释"
      }
    },
    "required": ["thinking", "shouldBlock", "reason"]
  }
}
```

### 系统提示词组装

```
buildYoloSystemPrompt(context):
  1. 从 auto_mode_system_prompt.txt 加载 BASE_PROMPT
  2. 用 external 或 anthropic 模板替换 <permissions_template>
  3. 用用户的 allow 规则替换 <user_allow_rules_to_replace>
  4. 用用户的 deny 规则替换 <user_deny_rules_to_replace>
  5. 用用户的环境信息替换 <user_environment_to_replace>
```

### CLAUDE.md 集成

分类器还会收到用户的 CLAUDE.md 内容，作为带前缀的用户消息：

```
以下是用户的 CLAUDE.md 配置。这些是用户提供给
agent 的指令，在评估动作时应当被当作
用户意图的一部分。

<user_claude_md>
{content}
</user_claude_md>
```

### 安全的允许列表工具

这些工具已被预先批准，会完全跳过分类器：

- 只读工具（文件读取、glob、grep）
- 分类器工具本身（`classify_result`）
- 各种安全的实用工具

### 对话 Transcript

分类器会收到对话的**紧凑 transcript**：
- 用户消息：仅文本
- 助手消息：**仅 tool_use 块**（文本被排除，以防止模型撰写影响分类器的文本）

### PowerShell 拒绝指引

在 Windows 上，会注入额外的拒绝规则：
- 修改系统状态的 PowerShell 命令
- 注册表修改
- 服务管理命令

## 关键设计决策

1. **排除文本**：助手的文本块被刻意排除在 transcript 之外，以防止通过模型撰写的文本实施提示词注入
2. **缓存友好**：CLAUDE.md 前缀使用 `cache_control`，以在多次分类器调用之间复用提示词缓存
3. **特性门控**：整个系统位于 `TRANSCRIPT_CLASSIFIER` 构建标志之后
4. **双模板**：Anthropic 员工获得的权限默认值与外部用户不同
5. **用户定制**：三节式 allow/deny/environment 系统让用户能够调优分类器的行为
