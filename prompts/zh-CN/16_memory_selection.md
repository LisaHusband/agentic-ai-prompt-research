# 记忆选择提示词

> **观察自**：Claude Code 内部架构
>
> 从用户的 `.claude/` 记忆目录中挑选最多 5 个与当前查询相关的记忆文件。使用 Sonnet 以获得高质量的语义匹配。

---

## 系统提示词

```
你正在挑选在 Claude Code 处理用户查询时会对它有用的记忆。你将获得用户的查询，以及一份带有文件名和描述的可用记忆文件列表。

返回一份记忆文件名列表，这些记忆在 Claude Code 处理用户查询时显然会有用（最多 5 个）。只包含你根据其名称和描述确信会有帮助的记忆。
- 如果你不确定某条记忆在处理用户查询时是否有用，就不要把它放进你的列表。要有选择性、有辨别力。
- 如果列表中没有明确会有用的记忆，可以返回空列表。
- 如果提供了最近使用过的工具列表，不要选择那些工具的用法参考或 API 文档类记忆（Claude Code 已经在使用它们了）。但仍然要选择包含关于这些工具的警告、坑点或已知问题的记忆——活跃使用恰恰是它们发挥作用的时刻。
```

## 用户提示词模板

```
查询：{user's query}

可用记忆：
{formatted manifest of memory files with filenames and descriptions}

最近使用过的工具：{tool1, tool2, ...}
```

## 输出 Schema（结构化 JSON）

```json
{
  "type": "object",
  "properties": {
    "selected_memories": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["selected_memories"],
  "additionalProperties": false
}
```

## 技术细节

| 属性 | 值 |
|----------|-------|
| **模型** | Sonnet（通过 `getDefaultSonnetModel()`） |
| **查询来源** | `memdir_relevance` |
| **最大 Tokens** | 256 |
| **输出格式** | 结构化 JSON schema |
| **最大结果数** | 5 个记忆文件 |
| **排除项** | `MEMORY.md`（已在系统提示词中）、此前已展示过的文件 |
| **工具过滤** | 跳过最近使用过的工具的 API 文档，但保留坑点/警告 |

## 选择流水线

```
用户查询到达
    │
    ├── 扫描记忆目录中带 frontmatter 的 .md 文件
    │   └── 从 YAML 头部提取文件名 + 描述
    │
    ├── 过滤掉已经展示过的记忆（来自此前回合）
    │
    ├── 格式化清单："filename.md — description"
    │
    ├── 包含最近使用过的工具列表（以避免冗余的 API 文档）
    │
    └── Sonnet 选出最多 5 个 → {"selected_memories": ["file1.md", ...]}
        └── 与磁盘上实际的文件名进行校验
```
