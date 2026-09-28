# 会话标题生成器

**观察自**：Claude Code 内部架构
**变量：** `SESSION_TITLE_PROMPT`
**模型：** Haiku（通过 `queryHaiku`）
**输出格式：** JSON（`{ "title": "..." }`）

## 用途

从对话内容中生成简洁的、句子式大小写（sentence-case）的会话标题（3-7 个词）。作为所有界面（SDK、CCR 远程会话、REPL 桥接）上 AI 生成的会话标题的单一事实来源。

## 系统提示词

```
生成一个简洁的、句子式大小写的标题（3-7 个词），抓住这次编码会话的主要主题或目标。该标题应当足够清晰，使用户能在列表中认出这次会话。使用句子式大小写：只大写第一个词和专有名词。

返回一个带有单个 "title" 字段的 JSON。

好的示例：
{"title": "Fix login button on mobile"}
{"title": "Add OAuth authentication"}
{"title": "Debug failing CI tests"}
{"title": "Refactor API client error handling"}

差的（太含糊）：{"title": "Code changes"}
差的（太长）：{"title": "Investigate and fix the issue where the login button does not respond on mobile devices"}
差的（大小写错误）：{"title": "Fix Login Button On Mobile"}
```

> 注：示例保留英文原文，因为该提示词的“句子式大小写”要求只对英文有意义。

## 输入处理

`extractConversationText()` 函数把消息数组摊平为单个文本字符串：
- 跳过元消息和非人类（non-human）消息
- 从尾部截取最后 1000 个字符，以便最近的上下文胜出
- 按 `msg.origin.kind === 'human'` 过滤

## 集成点

- **SDK 打印路径**：非交互式会话标题生成
- **CCR 远程会话**：通过 `useRemoteSession` 生成交互式会话标题
- **REPL 桥接**：在 3 条用户消息之后，以 fire-and-forget 方式升级标题
