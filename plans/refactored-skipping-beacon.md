# 竞品对标升级计划：AI 漫剧生成工具

## 背景

基于竞品"纳米漫剧流水线"（namistory）的深度分析，当前系统与其存在显著差距。竞品采用**四层架构**和**一体化流水线**，而我们的系统仍处于 MVP 阶段。

---

## 竞品分析要点

### 竞品四层架构
1. **用户层**：剧本编辑 → 分镜编辑 → 非线性编辑 → 视频编辑
2. **智能体层**：纳米蜂群智能体动态调度，协同约束 AI 发散性
3. **资产层**：角色、场景等固定资产管理，保证全局一致性
4. **生成层**：专属视频世界模型（Seedance 2.0）

### 竞品工作流
- 剧本解析 → 资产生成 → 分镜制作 → 动态合成

### 竞品核心优势
- 制作速度是主流工具 3 倍以上
- 单集生产时间 30-60 分钟
- 素材直出成功率 90%+
- 100% 创作可控（自然语言调整光影、材质、机位）

---

## 当前系统状态

### 前端（client）
| 页面/组件 | 现状 |
|----------|------|
| `/create` | 基础模板选择 + 场景编辑 |
| `/work/:id` | 基础作品详情 + 导出 |
| Dashboard | 基础作品列表 |
| TemplateSelector | 5 种画风 + 8 种风格 |
| SceneEditor | 基础多场景编辑 |

### 后端（server）
| 数据表 | 现状 |
|--------|------|
| User | 基础用户信息 |
| Work | 基础作品（标题、脚本、图片、音频、视频） |

### 差距分析
1. **无剧本结构化数据**：竞品有剧本解析，我们只有纯文本
2. **无分镜概念**：竞品有分镜制作，我们直接生成图片序列
3. **无资产层**：竞品有角色/场景资产库，我们无复用
4. **无智能调度**：竞品有蜂群智能体，我们只是简单调用 API

---

## 升级方案

### 第一阶段：数据模型扩展（优先级 P0）

#### 1.1 新增剧本表（Script）
```prisma
model Script {
  id          String   @id @default(uuid())
  workId      String
  work        Work     @relation(fields: [workId], references: [id], onDelete: Cascade)
  title       String
  content     String   @db.Text        // 原始剧本文本
  parsedData  Json?                  // AI 解析后的结构化数据
  scenes      Scene[]                // 关联的分镜
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

#### 1.2 新增分镜表（Scene）
```prisma
model Scene {
  id          String   @id @default(uuid())
  scriptId    String
  script      Script   @relation(fields: [scriptId], references: [id], onDelete: Cascade)
  order       Int                      // 分镜序号
  description String                    // 分镜描述
  prompt      String?                   // AI 图像生成提示词
  imageUrl    String?                   // 生成的图片
  audioUrl    String?                   // 配音
  duration    Float?                    // 持续时间（秒）
  camera      String?                   // 镜头参数（机位）
  lighting    String?                   // 光影设置
  assets      SceneAsset[]              // 场景内使用的资产
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

#### 1.3 新增资产表（Asset）
```prisma
model Asset {
  id          String    @id @default(uuid())
  userId      String
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  type        AssetType               // CHARACTER, SCENE, PROP
  name        String
  imageUrl    String
  description String?
  metadata    Json?                   // 扩展属性（风格、配色等）
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  sceneAssets SceneAsset[]
}

enum AssetType {
  CHARACTER  // 角色
  SCENE      // 场景
  PROP       // 道具
}

model SceneAsset {
  id        String  @id @default(uuid())
  sceneId   String
  scene     Scene   @relation(fields: [sceneId], references: [id], onDelete: Cascade)
  assetId   String
  asset     Asset   @relation(fields: [assetId], references: [id], onDelete: Cascade)
  role      String? // 角色在分镜中的角色（主角、配角）
}
```

#### 1.4 扩展 Work 表
```prisma
model Work {
  // ... 现有字段
  scripts     Script[]   // 支持多集剧本
  assets      Asset[]    // 使用的资产
}
```

---

### 第二阶段：前端工作流重构（优先级 P0）

#### 2.1 创建剧本编辑器（ScriptEditor）
```
位置: client/src/components/ScriptEditor.tsx
功能:
- 文本输入剧本
- AI 自动解析剧本（提取角色、场景、动作）
- 可视化剧本大纲
- 分镜自动拆分
```

#### 2.2 创建分镜编辑器（ShotEditor）
```
位置: client/src/components/ShotEditor.tsx
功能:
- 分镜列表管理
- 每个分镜：描述、提示词、配音、镜头参数
- 支持拖拽排序
- 自然语言调整光影/机位
```

#### 2.3 创建资产库（AssetLibrary）
```
位置: client/src/components/AssetLibrary.tsx
功能:
- 角色库管理（上传/AI生成角色）
- 场景库管理
- 资产复用（拖拽到分镜）
- 资产全局一致性保证
```

#### 2.4 更新 Create 页面工作流
```
新流程:
1. 剧本输入 → AI 解析 → 自动拆分分镜
2. 分镜精细调整（镜头、光影、配音）
3. 资产选择（角色、场景）
4. 批量生成（图片 + 语音）
5. 预览 + 导出
```

---

### 第三阶段：后端智能调度（优先级 P1）

#### 3.1 剧本解析 API
```
POST /api/parse/script
功能:
- 输入剧本文本
- 输出结构化数据（角色列表、场景列表、情绪曲线）
- 自动拆分建议分镜
```

#### 3.2 批量生成 API
```
POST /api/generate/batch
功能:
- 批量生成分镜图片
- 并发控制（避免 API 限流）
- 进度回调
- 错误自动重试
```

#### 3.3 资产一致性保证
```
功能:
- 提取角色特征向量
- 场景风格一致性检查
- 自动匹配相似资产
```

---

### 第四阶段：高级功能（优先级 P2）

| 功能 | 描述 |
|------|------|
| 非线性编辑 | 调整分镜顺序、时长 |
| 镜头参数 | 机位、运动、光影微调 |
| 角色一致性 | 多分镜同角色保持一致 |
| 批量导出 | 批量生成 MP4/GIF |

---

## 实施计划

### Sprint 1：数据模型
- [ ] 创建 Script、Scene、Asset 数据表
- [ ] 编写迁移脚本
- [ ] 更新后端 CRUD 接口

### Sprint 2：前端基础
- [ ] 创建 ScriptEditor 组件
- [ ] 创建 ShotEditor 组件
- [ ] 更新 Create 页面工作流

### Sprint 3：资产层
- [ ] 创建 AssetLibrary 组件
- [ ] 角色上传/生成功能
- [ ] 资产复用逻辑

### Sprint 4：智能调度
- [ ] 剧本解析 API
- [ ] 批量生成逻辑
- [ ] 进度通知

---

## 关键文件

### 需要修改
- `server/prisma/schema.prisma` - 数据模型
- `server/src/routes/works.ts` - 作品接口
- `server/src/routes/generate.ts` - 生成接口
- `client/src/pages/Create.tsx` - 创建页
- `client/src/App.tsx` - 路由

### 需要新增
- `server/src/routes/script.ts` - 剧本接口
- `server/src/routes/assets.ts` - 资产接口
- `client/src/components/ScriptEditor.tsx`
- `client/src/components/ShotEditor.tsx`
- `client/src/components/AssetLibrary.tsx`

---

## 验证方式

1. **数据模型**：运行 `npx prisma migrate dev`，验证表创建成功
2. **API 接口**：使用 Postman 测试 CRUD 接口
3. **前端页面**：访问 `/create`，验证新工作流可用
4. **端到端**：完整创建流程（剧本 → 分镜 → 资产 → 生成）
