# macOS Liquid Glass 桌面小组件开发规范

适用于原生 `AppKit`（Objective-C++ `.mm` 动态库 + C# P/Invoke）构建的 macOS 桌面级悬浮组件。

## 1. 窗口层级与桌面常驻行为

- **面板基类**：无边框 `NSPanel`（`NSWindowStyleMaskBorderless | NSWindowStyleMaskNonactivatingPanel`）。
- **核心属性**：
  - `opaque = NO`、`backgroundColor = [NSColor clearColor]`
  - `level = -2147483602`（常驻桌面壁纸层之上、普通应用窗口之下，不遮挡日常工作区）
  - `hidesOnDeactivate = NO`（应用失焦时不隐藏）
  - `movableByWindowBackground = NO`（禁用系统自带背景拖拽，改用自定义防误触状态机）
  - `collectionBehavior = NSWindowCollectionBehaviorCanJoinAllSpaces | NSWindowCollectionBehaviorStationary | NSWindowCollectionBehaviorIgnoresCycle`（全桌面空间可见、调度中心静止、不参与 `Cmd+Tab` / `Cmd+\`` 切换循环）
- **坐标持久化**：监听 `NSWindowDidMoveNotification` 实时保存窗口原点坐标，启动时读取并做屏幕可见区域夹紧（Clamp）。

## 2. 玻璃材质、阴影处理与光效细节

- **禁用系统黑影（关键）**：
  - `gPanel.hasShadow = NO`
  - `NSGlassEffectView` 与内部选中胶囊均设置 `layer.shadowOpacity = 0.0f`
  - 原因：原生 `Liquid Glass` / 毛玻璃开启系统阴影会在玻璃半透明边缘留下发灰脏边，去掉黑影后仅靠高光描边与内部对比度区分层次，通透感更强。
- **双版本玻璃底板适配**：
  - `macOS 26.0+`：使用 `NSGlassEffectView`（`style = NSGlassEffectViewStyleClear`）。
  - 旧版 macOS 降级：`NSVisualEffectView`（`blendingMode = NSVisualEffectBlendingModeBehindWindow`、`material = NSVisualEffectMaterialHUDWindow`、`state = NSVisualEffectStateActive`）。
- **1px 棱边高光（Specular Rim Highlight）**：
  - 顶层 `CAGradientLayer`（`zPosition = 999.0`，`opacity = 0.88f`）配合 `1px` 环形 `CAShapeLayer` mask。
  - 渐变色：纯白 `0.78` → 冰蓝 `(0.61, 0.88, 1.0, 0.34)` → 纯白 `0.06`，随鼠标移动实时微调高光重心。
- **点击激活焦散光晕（Caustics Glow）**：
  - `320×320` 径向 `CAGradientLayer`（`zPosition = 997.0`），默认 `opacity = 0.0f`（平时不打扰桌面）。
  - 仅在 `mouseDown` 按下/交互时设 `causticsActive = YES`、`opacity = 1.0f` 并跟随光标；`mouseUp` 后通过 `0.25s` 动画淡出至 `0.0f`。

## 3. 防误操作（防误触拖动与点击隔离）

- **顶部区域长按拖拽**：
  - 在根视图重写 `mouseDown:` / `mouseDragged:` / `mouseUp:`，仅当按下位置位于顶部 Header 非按钮区且持续按住 **`0.25s`** 后才进入拖拽态，移动窗口。
  - 轻点或鼠标快速划过不触发拖动，彻底避免日常操作把桌面组件误拖走。
- **按钮区严格隔离**：
  - 交互按钮（连接、降噪 4 档胶囊、坐姿开关、坐姿校准）拥有独立命中区域，点击立即响应业务回调，不冒泡触发拖拽。

## 4. 动画与控件规范

- **胶囊滑动切换**：降噪模式（如 `Off / Adaptive / ANC / Ambient`）使用轨道容器 + 滑动选中胶囊（圆角 `14.0`，外框轨道圆角 `18.0`），切换时平移动画更新。
- **连接入场动效**：设备连接成功时播放短暂 Hero 中心图标 + 绿色扩散圆环波纹（`gHeroRipple`）+ 状态提示文字，随后淡出恢复常规看板。
- **多语言同步**：所有硬编码文本（`Connect`、`Case`、模式标签、Tooltip 等）随 C# 端 `Settings.Data.Locale` 实时刷新，默认英文（`en`），支持一键切换为简体中文（`zh`）。
