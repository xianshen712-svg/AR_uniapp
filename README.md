# AR 虚拟现实 - UniApp 微信小程序

基于 UniApp + Vue2 开发的微信小程序 AR 应用，集成微信官方 `xr-frame` 能力，提供多种 AR/VR 场景演示。

## 项目简介

本项目是一个功能丰富的 AR/VR 微信小程序，基于微信官方的 `xr-frame` 框架开发，提供了多种 AR 能力的演示和体验。

## 核心功能

- **XR 能力卡片列表**：将 xr-frame 不同核心能力做成独立卡片展示
- **多种 AR 场景**：
  - 基础 3D 场景
  - AR 平面检测
  - AR 图片 Marker 识别
  - 人脸/手势追踪
  - glTF 模型加载
  - 物理引擎模拟
  - 视频纹理
  - 2D 追踪
  - 手势动画
  - 模型动画
  - OSD Marker
  - 手势产品交互
  - 光照效果

## 技术栈

- **框架**：UniApp + Vue2
- **平台**：微信小程序
- **AR 引擎**：微信官方 xr-frame
- **开发工具**：HBuilderX + 微信开发者工具

## 项目结构

```
ar虚拟现实/
├── xrPackage/           # XR 功能包
│   └── pages/          # XR 场景页面
├── pocketPackage/      # 口袋功能包
│   └── pages/          # 口袋页面
├── printPackage/       # 打印功能包
│   └── pages/          # 打印页面
├── static/             # 静态资源
│   └── models/         # 3D 模型文件
├── uni_modules/        # UniApp 插件模块
├── pages.json          # 页面配置
├── manifest.json       # 应用配置
└── App.vue             # 应用入口
```

## 快速开始

### 环境要求

- Node.js >= 14.0.0
- HBuilderX 或 VS Code
- 微信开发者工具

### 安装依赖

```bash
npm install
```

### 开发

1. 使用 HBuilderX 打开项目
2. 运行到微信小程序
3. 在微信开发者工具中预览

### 构建

```bash
# 构建微信小程序
npm run build:mp-weixin
```

## 功能演示

### XR 能力列表

点击首页的 "XR-Frame AR" 按钮进入能力卡片列表页，选择不同的 AR 能力进行体验。

### 场景导航

首页 → XR入口按钮 → 能力卡片列表页 → 点击卡片 → 对应XR场景演示页 → 可返回列表

## 配置说明

### 微信小程序配置

在 `project.config.json` 中配置你的小程序 AppID：

```json
{
  "appid": "your-appid"
}
```

### XR-Frame 配置

确保在 `manifest.json` 中启用了 xr-frame 相关权限。

## 注意事项

- 本项目基于微信小程序 xr-frame 能力，需要在真机上测试
- 部分 AR 功能需要设备支持相应的传感器
- 3D 模型文件较大，建议使用 CDN 加速

## 许可证

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！

## 联系方式

如有问题，请提交 Issue 或联系开发者。