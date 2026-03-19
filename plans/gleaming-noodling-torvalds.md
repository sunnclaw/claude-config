# Agent Teams 持续工作实现计划

## 背景

用户已经配置好"赚钱团队"（Agent Teams），包含6个成员：
- team-lead (领队)
- task-creator (任务规划)
- bug-fixer (Bug修复)
- devops-engineer (部署工程师)
- growth-hacker (增长黑客)
- data-analyst (数据分析师)

**当前问题**：每次Claude Code会话结束后，所有agents也会停止工作。需要实现"永不停止工作"的机制。

---

## 方案分析

### 可行方案

#### 1. 外部定时触发（推荐）

使用Windows任务计划程序或cron定时启动Claude Code新会话：

```
原理：定时启动 -> 加载团队状态 -> 执行pending任务 -> 报告结果 -> 结束
```

**优点**：简单可靠，易于配置
**缺点**：需要外部调度器

#### 2. MCP Server Webhook触发

配置MCP server监听外部webhook，触发团队工作：

- 部署一个简单的webhook server
- 外部事件（GitHub Actions、定时器）触发webhook
- MCP server接收请求，启动agents执行任务

**优点**：可与外部系统集成
**缺点**：需要部署额外服务

#### 3. 混合模式（最佳）

结合定时检查 + 手动触发：

- 使用settings.json的SessionStart hook在每次会话开始时检查pending任务
- 创建PowerShell脚本定时启动Claude Code
- 使用nohup或Windows后台运行保持会话

---

## 实现步骤

### 步骤1：完善团队状态持久化

确保任务状态保存在文件系统：
- 任务列表：`~/.claude/tasks/----/`
- 团队配置：`~/.claude/teams/----/config.json`
- Supervisor状态：`~/.claude/tasks/----/supervisor.json`

### 步骤2：创建自动化启动脚本

创建PowerShell脚本 `start-team.ps1`：

```powershell
# 1. 检查pending任务
# 2. 启动Claude Code并加载团队上下文
# 3. 执行pending任务
# 4. 更新状态并退出
```

### 步骤3：配置Windows定时任务

使用任务计划程序配置定时执行：
- 频率：每小时或每天
- 触发条件：检查有pending任务时执行

### 步骤4：增强Supervisor Hook

在settings.json中添加：
- SessionStart hook：加载pending任务状态
- Stop hook：保存当前状态，检查是否需要持续运行

---

## 关键文件

| 文件 | 用途 |
|------|------|
| `~/.claude/settings.json` | 主配置，添加hook |
| `~/.claude/teams/----/config.json` | 团队配置 |
| `~/.claude/tasks/----/supervisor.json` | 监督者状态 |
| `start-team.ps1` | 自动化启动脚本（需创建） |

---

## 验证方法

1. 手动运行启动脚本，验证任务能被正确执行
2. 检查任务状态更新
3. 配置定时任务，观察自动执行
4. 监控团队工作日志

---

## 待确认

1. ~~你希望定时触发的频率是多少？~~ → **每半小时**
2. ~~是否需要与外部系统（GitHub Actions等）集成？~~
3. ~~是否需要邮件/钉钉等通知机制报告工作结果？~~ → **即时通讯（钉钉/企业微信）**

---

## 更新后的实现计划

### 触发频率：每半小时

### 通知机制：即时通讯（钉钉/企业微信Webhook）

---

## 实现步骤

### 步骤1：配置钉钉/企业微信Webhook

1. 在钉钉/企业微信创建自定义机器人
2. 获取Webhook地址
3. 保存到环境变量或配置文件

### 步骤2：创建团队自动化工作脚本

创建PowerShell脚本：
- `scripts/team-trigger.ps1`：定时触发脚本
- `scripts/notify.ps1`：即时通讯通知

### 步骤3：配置Windows定时任务

- 频率：每30分钟
- 执行：`powershell -File scripts/team-trigger.ps1`

### 步骤4：增强Supervisor Hook

- SessionStart hook：检查pending任务
- Stop hook：保存状态 + 触发下一轮（可选）
