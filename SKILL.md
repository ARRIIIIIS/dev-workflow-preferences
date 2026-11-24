---
name: dev-workflow-preferences
description: >-
  Applies the user's personal development, UI/UX, macOS build/signing, and GitHub
  open-source/GEO workflows. Use when building macOS desktop widgets or utility apps,
  designing web settings wizards, publishing .NET macOS arm64 apps, or open-sourcing
  and optimizing repositories on GitHub. (用户个人开发习惯、macOS 打包签名、UI 偏好与 GitHub GEO 发布规范)
---

# 个人开发习惯与工作流规范

在开发 macOS 桌面应用、Web 设置向导或进行 GitHub 开源发布时，严格遵循以下规范与细节文档。

## 1. 沟通与执行风格

- **语言与表达**：使用**简体中文**回复，精简直奔结果，**不使用 emoji**。
- **先方案再动手**：面对开放性需求（优化、架构调整、GEO、重构等），先列出具体方案供确认，再开始修改；收到明确指令后直接执行到底（修改、编译、安装、签名、运行校验）。
- **风格复用**：设计新界面或向导页时，优先复用既有项目已验证的视觉体系，不另起炉灶。

## 2. macOS 桌面小组件规范

详见 [macOS Liquid Glass 桌面小组件规范](./references/macos-widget-spec.md)，核心要点：
- **桌面底层常驻**：无边框 `NSPanel`（`level = -2147483602`、`CanJoinAllSpaces | Stationary | IgnoresCycle`、`hidesOnDeactivate = NO`），不抢焦点、不遮挡普通窗口；拖动后持久化坐标并做屏幕边界夹紧。
- **防误操作**：禁用 `movableByWindowBackground`，非按钮区必须**长按 `0.25s`** 才触发拖动；按钮区与拖拽区严格隔离。
- **玻璃质感与阴影**：**禁用系统黑影**（`hasShadow = NO`、`layer.shadowOpacity = 0.0f`），避免玻璃边缘发灰脏边；使用 `NSGlassEffectView`（macOS 26+）/ `NSVisualEffectView`（降级），配合 **`1px` 棱边高光（`rimGrad`）** 与 **点击激活焦散光晕（`causticsLayer`）**。
- **极简托盘**：菜单栏仅保留 3 项（`Settings Wizard...`、`Restart`、`Quit`），默认英文，支持实时切换简体中文。

## 3. 工业风 Web 设置向导规范

详见 [工业风 Web 设置向导规范](./references/web-workbench-spec.md)，核心要点：
- **深色工业风切角美学**：`#060609` 极暗背景 + `72px` 透视网格 + 顶部渐变光晕；容器与按钮使用 **`clip-path` 斜切倒角（chamfer）**，荧光绿 `#c1ff12` 主强调色，`SF Mono` + `tabular-nums`。
- **布局与实时生效**：居中 `660px` 单栏工作台，顶部 Hero + 右上角 `EN | 中文` 切换 + 4 格 **HUD 遥测条**，下方为 **`01 / 02 / 03` 分步 Stepper**（标题 `01 // SECTION`）。
- **所见即所得**：型号/配色联动产品实物渲染预览图；开关、模式胶囊、滑块操作后**直接实时下发生效**，不设多余保存按钮。
- **曲线可视化（EQ）**：**9 段垂直柱状图 + SVG 三次贝塞尔平滑曲线**双层联动，拖拽实时重绘、松手立即下发。

## 4. macOS 构建签名与 GitHub GEO 发布

详见 [macOS 构建签名与 GitHub GEO 规范](./references/macos-build-and-geo.md)，核心要点：
- **单架构发布**：使用 `$HOME/.dotnet-sdk10` 仅发布 `-r osx-arm64`（避免 Universal 构建在 macOS 26/27+ 触发 `SIGKILL`）。
- **图标与防重复**：保留 `AppIcon.icns`（更新 `CFBundleIconFile`），清理 `bin/` 与 `obj/` 下的中间 `.app` 防止聚焦搜索重复。
- **逐个签名**：先逐个 `codesign --force --sign -` 所有 `Contents/MonoBundle/*.dylib`，再签 `.app`，启动后用 `pgrep -fl` 验证存活。
- **GitHub GEO**：设置高信息密度 `description` 与 10+ `topics`；`README.md`（默认英文）+ `README.zh-CN.md` 含 TL;DR、对比表、支持型号全称清单、FAQ 与 `xattr -cr` 解锁提示；`HTTP/1.1` 稳定推送。
