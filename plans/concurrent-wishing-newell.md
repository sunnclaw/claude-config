# 24/7 Claude AI 助手实现计划

## 背景

用户希望 Claude Code 在 VS Code 中变成 24/7 可用的 AI 助手，具有：
- 持久会话（不会丢失上下文）
- 完整工具权限
- MCP 服务器集成

---

## 理解用户需求

用户想要在 VS Code 中随时调用 Claude，就像现在这样，但希望它：
- 保持持久连接
- 记忆之前的对话上下文
- 24/7 可用

**现状**：Claude Code 扩展已经在 VS Code 中运行，每次启动会恢复上次的会话。

**问题**：用户可能想要：
1. 更强的持久化（跨会话记忆）
2. MCP 服务器集成到 VS Code 扩展中

---

## 实现方案

### 方案：配置 Claude Code 扩展

当前 Claude Code 扩展已支持：
- 持久会话恢复
- 子 Agent 记忆（memory: user）

需要配置的选项：

1. **启用子 Agent 记忆** - 让 AI 记住项目知识
2. **配置 MCP 服务器** - 集成 GitHub、文件系统等
3. **使用 `--continue`** - 每次继续上次会话

---

## 具体配置

### 1. 配置 MCP 服务器

创建/编辑项目中的 `.mcp.json`：

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${PROJECT_PATH}"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### 2. 配置子 Agent 记忆

在 `.claude/agents/` 目录下的 agent 配置中添加：
```yaml
memory: user  # 跨会话持久记忆
```

### 3. 最佳实践

- 每次使用 `--continue` 或 `--resume` 继续会话
- 使用 tmux/screen 保持终端会话

---

## 验证方式

1. 检查 MCP 配置：`claude mcp list`
2. 测试会话恢复：使用 `--continue` 参数
3. 验证子 Agent 记忆工作

---

## 备选方案：Agent SDK 守护进程

如果以上不满足需求，可以构建自定义守护进程，通过 VS Code 终端调用。
