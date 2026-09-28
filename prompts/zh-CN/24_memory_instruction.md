# 记忆指令（CLAUDE.md 系统）

**观察自**：Claude Code 内部架构
**变量：** `MEMORY_INSTRUCTION_PROMPT`

## 用途

把所有已加载的 CLAUDE.md 记忆文件包裹进系统提示词的那条元指令。这一行确立了用户提供的指令相对于默认行为具有绝对的优先权。

## 提示词

```
代码库和用户指令如下所示。务必遵守这些指令。
重要：这些指令覆盖任何默认行为，你必须严格按原文遵循它们。
```

## 记忆文件加载顺序

文件按优先级倒序加载（越晚加载 = 优先级越高）：

1. **托管记忆**（`/etc/claude-code/CLAUDE.md`）——面向所有用户的全局指令
2. **用户记忆**（`~/.claude/CLAUDE.md`）——面向所有项目的私有全局指令
3. **项目记忆**（项目根目录中的 `CLAUDE.md`、`.claude/CLAUDE.md`、`.claude/rules/*.md`）——已提交到代码库
4. **本地记忆**（项目根目录中的 `CLAUDE.local.md`）——私有的项目专属指令

## 文件发现

- 用户记忆从 `~/.claude/` 加载
- 项目文件和本地文件通过从当前目录向上遍历至根目录来发现
- 越接近当前目录的文件优先级越高（越晚加载）
- 每个目录中都会检查 `CLAUDE.md`、`.claude/CLAUDE.md`，以及 `.claude/rules/` 中的所有 `.md` 文件

## @include 指令

记忆文件支持传递式文件包含：

- 语法：`@path`、`@./relative/path`、`@~/home/path` 或 `@/absolute/path`
- 只在叶子文本节点中生效（不在代码块内部）
- 通过跟踪已处理的文件来防止循环引用
- 不存在的文件会被静默忽略
- 最大包含深度：5
- 只允许文本文件扩展名（防止加载图片、PDF 等）

## Frontmatter 支持

记忆文件支持带 `paths` 字段的 YAML frontmatter，用于条件注入：

```yaml
---
paths:
  - src/components/**
  - "*.tsx"
---
```

带有 `paths` frontmatter 的文件，只有在当前活动文件匹配这些 glob 模式时才会被注入。

## 配置

- `MAX_MEMORY_CHARACTER_COUNT`：40000 个字符（每个文件的建议上限）
- `claudeMdExcludes` 设置：用于排除特定 CLAUDE.md 文件的 glob 模式
- `MAX_INCLUDE_DEPTH`：5 层传递式包含
- 记忆文件中的 HTML 注释在注入前会被剥离
