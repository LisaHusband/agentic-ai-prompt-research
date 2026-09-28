# Update Config 技能（/update-config）

**观察自**：Claude Code 内部架构
**注册：** `registerBundledSkill('update-config', ...)`
**可用性：** 所有用户

## 用途

管理 `settings.json` 配置文件，包括 hooks、权限数组和常规设置。为编辑 Claude Code 配置层级提供一个有引导的界面。

## 提示词（从二进制分析重建）

该提示词篇幅庞大（约 476 行），涵盖三个主要方面：

### 设置文件层级

```
设置从三个层级加载（优先级从高到低）：
1. 项目设置：.claude/settings.json（已提交到仓库）
2. 用户设置：~/.claude/settings.json（私有，适用于所有项目）
3. 企业设置：/etc/claude-code/settings.json（由管理员管理）

较低优先级的设置会被较高优先级的设置覆盖。
```

### Hook 系统

该技能记录了完整的 hook 生命周期：

```
Hooks 是在 Claude Code 生命周期中特定时点运行的命令：
- PreToolUse：在工具被执行之前运行。可以阻止该工具。
- PostToolUse：在工具完成后运行。可以修改结果。
- PreCompact：在对话压缩之前运行。
- PostCompact：在对话压缩之后运行。
- Notification：当 Claude 想要通知用户时运行。
- Stop：当 Claude 的回合结束时运行。

每个 hook 通过 stdin 收到一个 JSON payload，其中包含：
- session_id：当前会话 ID
- tool_name：工具的名称（PreToolUse/PostToolUse）
- tool_input：工具输入参数（PreToolUse/PostToolUse）
- tool_output：工具输出（仅 PostToolUse）

Hook 退出码：
- 0：成功，正常继续
- 2：阻止该工具（仅 PreToolUse）
- 其他：记录错误但继续
```

### 权限数组

```
权限设置控制哪些工具可以在没有用户批准的情况下运行：
- allowedTools：被自动批准的工具名称或模式
- deniedTools：始终被阻止的工具名称或模式
- trust：不同工具类别的信任级别
```

## 架构说明

- 该技能区分“简单”配置工具（键值设置）和“复杂”操作（hooks、权限）
- 对于 hook 编辑，直接在 settings.json 文件上使用 Edit 工具
- 每次编辑后校验 JSON 语法
- 在单一界面中支持全部三个设置层级
