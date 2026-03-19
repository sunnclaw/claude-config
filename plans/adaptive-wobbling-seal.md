# AI 漫剧项目 - 角色一致性功能实现计划

## 背景
目前项目生成的每帧漫画，人物脸孔都不一样，无法形成连贯的角色形象。需要实现角色管理模块，使同一角色生成的多张图像保持面部一致。

## 架构概述

### 技术方案
1. **角色管理**: 创建 Character 模型，存储角色描述和三视图参考图
2. **角色一致性**: 生成图像时，将角色参考图作为 IPix/Image-to-Image 的参考
3. **三视图生成**: 使用 AI 生成角色的正面、侧面、背面三视图

### 文件结构
```
server/
├── prisma/schema.prisma          # 添加 Character 模型
├── src/
│   ├── routes/characters.ts      # 新建: 角色 CRUD API
│   ├── routes/generate.ts        # 修改: 支持角色参考
│   └── index.ts                  # 修改: 注册路由

client/src/
├── api/client.ts                # 修改: 添加角色 API
├── pages/
│   ├── CharacterStudio.tsx      # 新建: 角色工作台页面
│   └── Create.tsx               # 修改: 添加角色选择器
└── App.tsx                      # 修改: 添加路由
```

## 实现步骤

### 第一步: 后端 - Prisma 模型 (5 min)
在 `server/prisma/schema.prisma` 添加 Character 模型：
```prisma
model Character {
  id            String   @id @default(uuid())
  userId        String
  user          User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  name          String
  description   String?  @db.Text
  age           String?
  gender        String?
  personality   String?  @db.Text
  appearance    String?  @db.Text
  outfit        String?  @db.Text
  frontImage    String?  // 正面图
  sideImage     String?  // 侧面图
  backImage     String?  // 背面图
  style         String?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}
```

更新 User 模型添加 characters 关联

### 第二步: 后端 - 角色 API (15 min)
创建 `server/src/routes/characters.ts`：
- `POST /api/characters` - 创建角色
- `GET /api/characters` - 获取角色列表
- `GET /api/characters/:id` - 获取角色详情
- `PUT /api/characters/:id` - 更新角色
- `DELETE /api/characters/:id` - 删除角色
- `POST /api/characters/:id/generate-image` - 生成三视图

### 第三步: 后端 - 图像生成增强 (10 min)
修改 `server/src/routes/generate.ts`：
- `/image` 和 `/batch` 接口接收 `characterReferences` 参数
- 在 prompt 中加入角色一致性提示词
- 调用 IPix 或 Image-to-Image API 时传入参考图

### 第四步: 前端 - API 客户端 (5 min)
修改 `client/src/api/client.ts`：
- 添加 `characters` API 接口

### 第五步: 前端 - 角色工作台 (20 min)
创建 `client/src/pages/CharacterStudio.tsx`：
- 角色列表展示
- 创建/编辑角色表单
- 一键生成三视图
- 角色预览

### 第六步: 前端 - Create 页面集成 (15 min)
修改 `client/src/pages/Create.tsx`：
- 在 Assets 步骤添加角色选择面板
- 角色选择后显示在生成流程中

### 第七步: 前端 - 路由 (5 min)
修改 `client/src/App.tsx`：
- 添加 `/characters` 路由指向 CharacterStudio

### 第八步: 数据库迁移 (5 min)
```bash
cd server
npx prisma migrate dev --name add_characters
```

## 关键实现细节

### 角色一致性技术方案
根据 AI-Comic-Generator 的实现方式：
1. **三视图生成**: 使用 AI 生成详细的角色描述，然后生成正面、侧面、背面三视图
2. **Image-to-Image**: 生成其他画面时，用角色图作为 IPix 参考
3. **Prompt 增强**: 在 prompt 中加入角色特征描述

### 生成三视图的 Prompt 模板
```
Generate a character reference sheet for a [style] style character.
Character description: [description]
Requirements:
- Front view (正面): Full body, facing camera, neutral expression
- Side view (侧面): Profile view, showing body contour
- Back view (背面): Full body, showing back details
- Consistent facial features across all views
- Outfit: [outfit description]
```

### 角色一致性 Prompt 增强
```
[original prompt]
Include character features: [name], [age], [gender], [appearance description], [outfit]
Maintain consistent face with reference images
```

## 验收标准
1. ✓ 可以创建、编辑、删除角色
2. ✓ 可以生成角色三视图形象
3. ✓ 在生成漫剧画面时可以选择使用角色参考
4. ✓ 同一角色生成的多张图像保持面部一致
5. ✓ 前端页面美观可用

## 验证方法
1. 运行 `npm run dev` 启动开发服务器
2. 访问 `/characters` 创建角色
3. 生成角色三视图
4. 在 Create 页面选择角色生成漫画
5. 验证同一角色生成的多张图像面部一致
