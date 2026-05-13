# XR-Frame 案例实现任务计划

## 项目概述
基于 UniApp + Vue2 + 微信小程序，使用云诺XRFrame插件实现XR-Frame典型案例。

## Git 工作流程

```
main (主分支，稳定版本)
  │
  ├── feature/basic-scene (基础3D场景) ✅
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/marker-video (AR扫描播放视频) ✅
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/tracker-2d (综合图片视频识别) ✅
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/hand-product (手部产品销售) ✅
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/gltf-dynamic (动态模型加载) ✅
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/lighting (光照效果) ✅
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/osd-marker (OSD物体识别)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/ar-share (AR截图分享)
  │   └── 开发完成 → PR → Merge
  │
  ├── feature/hand-animation (手部动画交互)
  │   └── 开发完成 → PR → Merge
  │
  └── feature/model-animation (模型动画系统)
      └── 开发完成 → PR → Merge
```

## 案例清单

### 第一阶段：基础案例 (已完成 ✅)
- [x] 基础3D场景 (xr-basic-scene) - 立方体、灯光、纹理、环境天空盒
- [x] AR扫描播放视频 (xr-marker-video) - 扫描Marker图片播放视频内容

### 第二阶段：AR识别案例 (部分完成 🔄)
- [ ] OSD物体识别 (xr-osd-marker) ⭐⭐⭐
- [ ] AR截图分享 (xr-ar-share) ⭐⭐⭐
- [x] 综合图片视频识别 (xr-tracker-2d) ⭐⭐⭐ - 扫描图片播放视频，支持分享

### 第三阶段：手部识别案例 (部分完成 🔄)
- [ ] 基础手部识别 (xr-hand-basic)
- [x] 手部产品销售 (xr-hand-product) ⭐⭐⭐ - AR虚拟试戴，手势交互购物
- [ ] 手部动画交互 (xr-hand-animation) ⭐⭐⭐

### 第四阶段：glTF模型案例 (部分完成 🔄)
- [ ] 头盔模型展示 (xr-gltf-helmet)
- [ ] 无光照模型 (xr-gltf-unlit)
- [x] 动态模型加载 (xr-gltf-dynamic) ⭐⭐⭐ - 运行时切换不同glTF模型

### 第五阶段：动画与特效 (部分完成 🔄)
- [ ] 模型动画系统 (xr-model-animation) ⭐⭐⭐
- [ ] 内置几何体 (xr-mesh-basic)
- [x] 光照效果 (xr-lighting) ⭐⭐⭐ - 5种光源类型，强度/颜色调节

### 第六阶段：高级案例 (待开发 ⏳)
- [ ] 粒子效果 (xr-particles) - 旋转云粒子效果
- [ ] Shadow DOM多模型 (xr-shadow-dom) - 动态增删模型
- [ ] 视频纹理 (xr-video-texture) - 视频作为材质贴图
- [ ] 地球交互 (xr-earth-touch) - 触摸旋转交互

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

---

## 进度追踪

| 阶段 | 案例 | 分支 | 状态 | 难度 | 完成日期 |
|------|------|------|------|------|----------|
| 1 | 基础3D场景 | feature/basic-3d | ✅ 完成 | ⭐ | - |
| 1 | AR扫描播放视频 | feature/video-texture | ✅ 完成 | ⭐⭐ | - |
| 2 | 综合图片视频识别 | feature/tracker-2d | ✅ 完成 | ⭐⭐⭐ | 2025-05-13 |
| 2 | OSD物体识别 | feature/osd-marker | ⏳ 待开发 | ⭐⭐⭐ | - |
| 2 | AR截图分享 | feature/ar-share | ⏳ 待开发 | ⭐⭐⭐ | - |
| 3 | 手部产品销售 | feature/hand-product | ✅ 完成 | ⭐⭐⭐ | 2025-05-13 |
| 3 | 手部动画交互 | feature/hand-animation | ⏳ 待开发 | ⭐⭐⭐ | - |
| 4 | 动态模型加载 | feature/gltf-dynamic | ✅ 完成 | ⭐⭐⭐ | 2025-05-13 |
| 5 | 光照效果 | feature/lighting | ✅ 完成 | ⭐⭐⭐ | 2025-05-13 |
| 5 | 模型动画系统 | feature/model-animation | ⏳ 待开发 | ⭐⭐⭐ | - |
| 6 | 粒子效果 | feature/particles | ⏳ 待开发 | ⭐⭐⭐ | - |
| 6 | Shadow DOM | feature/shadow-dom | ⏳ 待开发 | ⭐⭐⭐ | - |

---

## 当前进行中的任务

### 已完成的3星难度案例 (4个) ✅
1. **综合图片视频识别** (xr-tracker-2d) - AR扫描图片播放视频
2. **手部产品销售** (xr-hand-product) - AR虚拟试戴，手势交互
3. **动态模型加载** (xr-gltf-dynamic) - 运行时切换glTF模型
4. **光照效果** (xr-lighting) - 5种光源类型与参数调节

### 待开发的3星难度案例 (5个) ⏳
1. **OSD物体识别** (xr-osd-marker) - 识别现实物体
2. **AR截图分享** (xr-ar-share) - AR场景截图分享
3. **手部动画交互** (xr-hand-animation) - 手势触发动画
4. **模型动画系统** (xr-model-animation) - 复杂动画控制
5. **粒子效果** (xr-particles) - 旋转云粒子

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

## 下一步行动

**建议优先级：**

1. **AR截图分享** (feature/ar-share)
   - 实用性强，用户刚需
   - 技术难度适中
   - 可复用到其他AR场景

2. **手部动画交互** (feature/hand-animation)
   - 与手部产品销售形成系列
   - 展示手势识别能力
   - 交互体验好

3. **OSD物体识别** (feature/osd-marker)
   - 与2D Marker形成对比
   - 展示不同识别技术
   - 适合特定场景

准备好开始了吗？请告诉我！
