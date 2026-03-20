# 游戏开发经验总结

## 核心教训

### 1. 改触发逻辑 vs 改表现层

**问题现象**：当游戏叙事节奏/时序不对时，容易只在 UI 表现层做改动，而不动触发层。

**反面案例（本次教训）**：
- 目标：让"SystemPrompt 被 X 人格接管"有明确的节拍感
- 我的错误做法：在 `narrative_text.gd` 和 `x_personality_display.gd` 加了接管动画
- 实际问题：`intro_corridor.gd` 里同一个点同时触发两个信号，节拍本身没改
- 结果：效果比原来好，但叙事节拍不对

**正确做法**：
- 先改触发层的时序（`intro_corridor.gd`）
- 再改表现层的动画
- 两者都改才能形成正确的节奏

---

### 2. 手感参数 vs 演出参数

游戏参数分两类，修改时风险完全不同：

| 类型 | 例子 | 风险 | 修改方式 |
|------|------|------|----------|
| **手感参数** | 移动速度、跳跃高度、切换锁定时长 | 直接影响玩家操作手感 | 必须真机测试后才能定 |
| **演出参数** | 动画时长、颜色值、过渡效果 | 只影响视觉表现 | 可以直接改代码 |

**反面案例**：
- `world_manager.gd` 的 `transition_duration` = 0.5s
- 我改成了 1.2s，认为"反正只是动画"
- 但这个值同时控制了 `is_transitioning` 锁定时长 = 手感参数
- 没有真机测试就改了，可能影响操控感

**正确做法**：
- 演出参数：直接改
- 手感参数：先在代码里留可调参数，真机测试后定值
- 没条件真机测时，默认偏短不偏长（玩家对"慢"比"快"更敏感）

---

### 3. 先沉淀再动手

**问题现象**：收到任务后容易直接动手，不先总结经验教训。

**本次教训**：
- Codex 指出核心问题后，我直接开始改代码
- Codex 再次反馈：核心问题（触发层没改）没有解决
- 根本原因：没有先分析清楚"这次的问题在哪里"，就直接动手

**正确工作流**：
```
收到任务
  → 分析问题根因
  → 沉淀经验/教训（写进文档）
  → 再执行
```

---

## 项目特定经验：时之潮 (Tide of Time)

### 叙事 UI 四层体系

| 层级 | 名称 | 位置 | 视觉风格 | 职责 |
|------|------|------|----------|------|
| Tier 0 | PoemLayer | 全屏居中 | 黑底 + 淡灰字 | 章节诗句/真正的黑场时刻 |
| Tier 1 | (复用 PoemLayer) | 全屏居中 | 同上，更高透明度 | **高优先级叙事中断**（不是"更猛的全屏"） |
| Tier 2 | NarrativeBox | 底部 | 灰褐框 + 顶部细线 | 主体叙事文字 |
| Tier 3 | SystemPrompt | 右上角 | 小红褐框 | 系统提示/操作指引 |
| X人格 | XPersonalityDisplay | 右上角 | 与 Tier 3 共用位置 | X人格劫持系统提示位 |

**Tier 1 的正确理解**：
- 不是"更猛的全屏"
- 是"高优先级叙事中断"
- 设计上 Tier 1 使用频率低，但承担关键叙事节点

---

### 关键文件清单

| 文件 | 职责 | 类型 |
|------|------|------|
| `scripts/intro/intro_corridor.gd` | 初始甬道叙事触发逻辑 | **触发层** |
| `scripts/ui/narrative_text.gd` | 四层叙事 UI 渲染层 | 表现层 |
| `scripts/ui/x_personality_display.gd` | X 人格对话显示 | 表现层 |
| `scripts/ui/world_switch_feedback.gd` | 世界切换视觉反馈 | 表现层 |
| `scripts/autoload/narrative_manager.gd` | 叙事管理器，信号中枢 | 信号层 |
| `scripts/autoload/x_personality_manager.gd` | X 人格状态机，三阶段管理 | 状态层 |
| `scripts/autoload/world_manager.gd` | 世界切换 + `transition_duration` | 状态层 |

---

### 待修复项（截至 2026-03-19）

1. **~~X 人格接管节拍~~** ✅ 已修复：
   - 文件：`intro_corridor.gd` 第 86-102 行
   - 新增 `_trigger_x_line_delayed()` 方法，延迟 0.3s 触发 X
   - `_update_system_beats()` 中两处触发均已改为延迟模式

2. **世界切换手感**：
   - 文件：`world_manager.gd` 第 9 行 `transition_duration`
   - 已改为 0.75s（保守值）
   - 待真机测试确认手感是否合适

3. **Tier 1 覆盖**（设计选择）：
   - X 人格目前只劫持 Tier 3
   - Tier 1 是否也需要被 X 劫持？需要明确设计意图

---

## 一般性工程原则

### 调试游戏叙事问题时

```
问自己：
1. 触发逻辑对吗？（发生在哪一行/哪个函数）
2. 表现层对吗？（动画/UI 渲染对吗）
3. 两者是否同步？（触发的时候，表现层准备好了吗）
```

### 修改游戏手感参数时

```
步骤：
1. 先确认哪个参数是手感参数（影响输入响应）
2. 留成可调参数，不要 hardcode
3. 真机测试后才定值
4. 没条件真机测 → 默认偏短不偏长
```

### 分析代码时

```
重要：
- 不要只读代码表面，要理解设计意图
- 空间位置（右上/右下）要核对清楚
- 理解参数的实际用途再改
```

---

*最后更新：2026-03-20*
*来源：时之潮叙事 UI 打磨项目 + 甬道扩展 auto-run*

---

### 4. 甬道叙事节奏：叙事长度必须匹配场景长度

**问题现象**：
- 玩家反馈：还没播完叙事就到终点了
- 根本原因：叙事内容耗时 > 玩家穿越甬道时间

**分析维度**：
| 维度 | 公式 | 关键参数 |
|------|------|----------|
| 穿越时间 | 距离 / 速度 | 场景长度、玩家速度 |
| 叙事时间 | 对话数量 × 平均对话时长 | 触发点数量、每句显示时长 |

**本次教训**：
- 1280px 甬道、6句叙事 → 叙事太长，跑到终点
- 扩展到 2800px 后仍不够 → auto_run 演出接管控制权
- 教训：先算好时间，再动手扩场景

**正确做法**：
```
1. 估算叙事总时长（所有对话 + 过渡）
2. 估算玩家穿越时长（距离 / 速度）
3. 如果叙事 > 穿越 → 需要减速/扩场景/加叙事
4. 如果叙事 << 穿越 → 可以加密叙事点或拉长演出
```

---

### 5. X 人格接管：必须真正接管玩家输入

**问题现象**：
- X 说"我来帮你按方向键"，但玩家仍然可以乱走
- `auto_run: true` 触发了，但玩家还能控制方向

**根本原因**：
- `intro_corridor.gd` 里设置了 `_x_auto_running = true`
- 但玩家的 `_handle_movement()` 没有检查这个状态
- 需要在 `player.gd` 添加 `allow_movement` 标志

**正确做法**：
```gdscript
# player.gd
@export var allow_movement: bool = true

func _handle_movement() -> void:
    if not allow_movement:
        return
    # 正常移动逻辑...
```

```gdscript
# intro_corridor.gd
func _start_x_auto_run() -> void:
    _x_auto_running = true
    player.allow_movement = false  # 真正接管

func _update_auto_run(delta: float) -> void:
    if not _x_auto_running or not player:
        return
    if player.allow_movement:  # 如果还没接管，跳过
        return
    player.velocity = Vector2(_auto_run_speed, 0)
    player.move_and_slide()
```

**教训**：
- 触发层说"接管" ≠ 真正接管
- 必须修改被接管对象的实际输入处理逻辑

---

### 6. 演出时序：auto_run + scene_transition 分离

**问题现象**：
- X 说"来吧我帮你按"，紧接着就说"我们到夏夜了"
- 节奏太紧密，没有演出感

**正确分离**：
```
x=1050: "让我陪你..." → auto_run 开始（X 接管控制权）
x=1350: "来吧..." → scene_transition 触发（视觉切换）
       ↓
       [玩家看着 X 带自己跑，1.5秒后场景变换]
x=1700: "或许你可以想象..." → 场景已经变换
```

**教训**：
- auto_run 和 scene_transition 不应在同一帧触发
- 留出"玩家被动观看"的演出时间
- 场景切换要有渐变过渡（1-1.5秒）

---

### 7. 调试游戏叙事：print() 是你的朋友

**Godot 没有日志文件**，但有 Output 面板。

**调试甬道叙事必备日志**：
```gdscript
# 关键状态变化
print("[Intro] Milestone reached: x=%.0f, queue_size=%d" % [x, _guide_queue.size()])
print("[Intro] Playing line: '%s', auto_run=%s" % [text, auto_run])
print("[Intro] Exit blocked - guide_queue=%d, next_line=%d/%d" % [...])

# 每秒位置报告
if int(player.global_position.x) % 100 < 2:
    print("[Intro] Player x=%.1f, auto_run=%s" % [player.global_position.x, _x_auto_running])
```

**日志分析问题**：
- 看到 `Exit blocked` → 叙事还没播完
- 看到 `auto_run=true` 但位置不变 → `_update_auto_run` 没被调用
- 看到位置到了但没 trigger → `_x_guidance_started` 是 false

---

### 8. 场景扩展策略：先放大再缩

**本次教训**：
- 不确定甬道应该多长 → 先扩到 2800px（2倍多）
- 不确定叙事点应该多密 → 先拉大间距
- 测试后根据实际情况收缩

**正确做法**：
```
怀疑某个值不够 → 直接×2或×3
跑一遍 → 确认"是太多了还是不够"
→ 收缩到合适值
```

好处：避免反复修改，每次都是确定性的"太多/不够"

---

### 9. Git 协作：本地修改要及时 commit

**问题现象**：
- Codex 在本地改了 43 个文件
- 用户不知道哪些是远程最新的，哪些是本地修改

**教训**：
- 开发前先 `git pull` 确认最新状态
- 每次完成一个可测试的改动就 commit
- commit message 要描述清楚改了什么

---

## 项目特定经验：时之潮 (TideOfTime) - 甬道篇

### 甬道叙事结构（截至 2026-03-20）

| 阶段 | 位置 | 事件 | 备注 |
|------|------|------|------|
| 引导 | x=40 | 玩家出生 | 速度=72 |
| 系统引导1 | x=0 | "请按下..." | 等待按键 |
| 系统引导2 | 按键后 | "请按气象键" | 等待按键 |
| 入侵序列 | 按键后 | 系统→X→警告→人类 | 全自动 |
| X引导1 | x=520 | "你好..." | 速度=42 |
| X引导2 | x=700 | "继续向右走" | |
| **auto_run** | x=1050 | "让我陪你..." | X接管控制 |
| **scene_transition** | x=1350 | "来吧..." | 夏夜氛围切换 |
| X引导3 | x=1700 | "或许你可以想象..." | 场景已变夏夜 |
| 白鸽 | x=2100 | "你瞧..." | 白鸽出现 |
| 结束 | x=2480 | "到家了" | |

**关键参数**：
- 场景宽度：2800px
- 出口位置：x=2503
- auto_run 速度：58.0
- guided_speed：42.0

---

*最后更新：2026-03-20*
*来源：甬道叙事扩展 + auto-run 演出功能*
