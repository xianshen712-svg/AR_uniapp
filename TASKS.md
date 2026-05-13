# XR-Frame 案例实现任务计划

## 项目概述
基于 UniApp + Vue2 + 微信小程序，使用云诺XRFrame插件实现XR-Frame典型案例。

## Git 工作流程

```
main (主分支，稳定版本)
  │
  ├── feature/basic-scene (基础3D场景)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/marker-video (AR扫描播放视频)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/osd-marker (OSD物体识别)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/ar-share (AR截图分享)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/hand-tracking (手部识别)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/gltf-model (glTF模型加载)
  │   └── 开发完成 → PR → Merge
  │
  └── feature/animation (模型动画)
      └── 开发完成 → PR → Merge
```

## 案例清单

### 第一阶段：基础案例 (已完成 ✅)
- [x] 基础3D场景 (xr-basic-scene)
- [x] AR扫描播放视频 (xr-marker-video)

### 第二阶段：AR识别案例
- [ ] OSD物体识别 (xr-osd-marker)
- [ ] AR截图分享 (xr-ar-share)
- [ ] 综合图片视频识别 (xr-tracker-2d)

### 第三阶段：手部识别案例
- [ ] 基础手部识别 (xr-hand-basic)
- [ ] 手部产品销售 (xr-hand-product)
- [ ] 手部动画交互 (xr-hand-animation)

### 第四阶段：glTF模型案例
- [ ] 头盔模型展示 (xr-gltf-helmet)
- [ ] 无光照模型 (xr-gltf-unlit)
- [ ] 动态模型加载 (xr-gltf-dynamic)

### 第五阶段：动画与特效
- [ ] 模型动画系统 (xr-animation)
- [ ] 内置几何体 (xr-mesh-basic)
- [ ] 光照效果 (xr-lighting)

## 每个案例的开发流程

### 1. 创建功能分支
```bash
git checkout main
git pull origin main
git checkout -b feature/案例名称
```

### 2. 开发步骤
- [ ] 创建页面目录和基础文件
- [ ] 配置 pages.json 路由
- [ ] 实现页面功能
- [ ] 使用云诺XRFrame插件
- [ ] 添加返回按钮
- [ ] 本地测试

### 3. 提交代码
```bash
git add .
git commit -m "feat: 添加XXX案例"
git push origin feature/案例名称
```

### 4. 创建PR合并
- 创建 Pull Request
- 代码审查
- 合并到 main 分支
- 删除功能分支

## 当前进行中的任务

### 任务 1: OSD物体识别案例
**分支**: `feature/osd-marker`
**状态**: 🔄 待开始
**预计耗时**: 2-3小时

**功能描述**:
- 使用OSD模式识别物体
- 识别成功后显示信息卡片
- 支持多个物体识别

**实现步骤**:
1. 创建 `xr-osd-marker` 页面
2. 使用 `<yn-ar>` 组件，设置 `yn_ar_mode="OSD"`
3. 配置多个识别目标
4. 实现信息展示UI
5. 测试并提交

---

## 开发规范

### 文件命名
- 页面目录: `xr-功能名称`
- 文件: `index.vue`, `index.json`

### 代码规范
- 使用 Vue2 语法
- 云诺插件组件: `<yn-viewer>`, `<yn-ar>`, `<yn-3d>`
- 返回按钮使用 `yn-viewer` 的 `showBackBtn` 属性

### 提交信息规范
```
feat: 添加新功能
fix: 修复问题
docs: 更新文档
style: 代码格式调整
refactor: 重构代码
```

---

## 进度追踪

| 阶段 | 案例 | 分支 | 状态 | 完成日期 |
|------|------|------|------|----------|
| 1 | 基础3D场景 | - | ✅ 完成 | - |
| 1 | AR扫描播放视频 | - | ✅ 完成 | - |
| 2 | OSD物体识别 | feature/osd-marker | 🔄 待开始 | - |
| 2 | AR截图分享 | feature/ar-share | ⏳ 待开始 | - |
| 2 | 综合识别 | feature/tracker-2d | ⏳ 待开始 | - |
| 3 | 手部识别 | feature/hand-basic | ⏳ 待开始 | - |
| 3 | 手部产品销售 | feature/hand-product | ⏳ 待开始 | - |
| 3 | 手部动画 | feature/hand-animation | ⏳ 待开始 | - |
| 4 | 头盔模型 | feature/gltf-helmet | ⏳ 待开始 | - |
| 4 | 无光照模型 | feature/gltf-unlit | ⏳ 待开始 | - |
| 4 | 动态模型 | feature/gltf-dynamic | ⏳ 待开始 | - |
| 5 | 动画系统 | feature/animation | ⏳ 待开始 | - |
| 5 | 几何体 | feature/mesh-basic | ⏳ 待开始 | - |
| 5 | 光照效果 | feature/lighting | ⏳ 待开始 | - |

---

## 下一步行动

1. **创建 feature/osd-marker 分支**
2. **实现 OSD物体识别案例**
3. **测试并合并到 main**

准备好开始了吗？请告诉我！
