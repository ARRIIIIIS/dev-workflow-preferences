---
name: dev-workflow-preferences
description: >-
  Applies the user's personal development, UI/UX, macOS build/signing, and GitHub
  open-source/GEO workflows. Use when building macOS desktop widgets or utility apps,
  designing web settings wizards, publishing .NET macOS arm64 apps, or open-sourcing
  and optimizing repositories on GitHub. (用户个人开发习惯、macOS 打包签名、UI 偏好与 GitHub GEO 发布规范)
---

# 个人开发习惯与工作流规范

在开发 macOS 桌面应用、Web 设置向导或进行 GitHub 开源发布时，遵循以下约定。

## 1. 沟通与执行风格

- **语言与表达**：使用**简体中文**回复，风格精简、直奔结果，**不使用 emoji**。
- **先沟通再动手**：面对开放性需求（如优化、架构调整、GEO、重构等），先列出具体方案供确认，再开始修改；收到明确指令后直接执行到底（修改、编译、安装、验证）。
- **视觉复用**：设计新界面或向导页时，优先参考用户已有项目（如工业风深色 HUD + 分步式网页工作台）保持风格统一。

## 2. 产品、UI 与多语言偏好

- **macOS 桌面端形态**：
  - 偏好原生 **AppKit Liquid Glass** 悬浮桌面组件（`NSVisualEffectView`、`1px` 棱边高光、点击焦散光晕反馈、顶部 `0.25s` 长按防误触拖动），搭配**极简 3 项菜单栏托盘**（设置向导、重启、退出）。
  - 复杂配置统一放在**本地 Web 设置向导（工作台）**中，包含 HUD 状态栏、分步导航、产品渲染预览图与即时生效控件。
- **数据可视化与交互**：
  - 音频/传感器曲线（如 EQ）偏好**柱状图 + 平滑曲线实时联动**，拖拽滑块或柱子时即时更新。
- **多语言默认策略**：
  - 桌面组件、菜单栏托盘、Web 向导及 `README.md` **默认使用英文（`en`）**，同时支持实时切换为**简体中文（`zh-CN`）**，且多端语言状态同步。

## 3. macOS 构建、图标、签名与运行校验

构建和安装 `.NET` macOS 桌面应用（Apple Silicon）时严格按以下流程处理：

1. **单架构发布（`osx-arm64`）**：
   - 使用 `$HOME/.dotnet-sdk10`（`DOTNET_ROOT=$HOME/.dotnet-sdk10`），**仅发布 `-r osx-arm64`**（避免 Universal `x86_64;arm64` 构建在较新 macOS 上触发 `SIGKILL (Code Signature Invalid)`）。
   ```bash
   dotnet publish <Project.csproj> -c Release -r osx-arm64 \
     -p:UseMacOSSDK=true -p:AllowMissingPrunePackageData=true -p:ValidateXcodeVersion=false
   ```
2. **保留应用图标与清理重复包**：
   - 将 `AppIcon.icns` 复制到 `Contents/Resources/AppIcon.icns`，并确保 `Info.plist` 中 `CFBundleIconFile` 为 `AppIcon.icns`。
   - 删除 `bin/` 和 `obj/` 目录下的中间 `.app` 包，避免聚焦搜索（Spotlight）和启动台出现重复图标。
3. **Ad-Hoc 本地签名**：
   - `codesign --deep` 会漏掉 `MonoBundle/*.dylib`，必须先逐个签名所有 `.dylib`，再签名整个 `.app`：
   ```bash
   for f in "/Applications/<App>.app/Contents/MonoBundle/"*.dylib; do
     codesign --force --sign - "$f" >/dev/null 2>&1
   done
   codesign --force --sign - "/Applications/<App>.app"
   ```
4. **启动与校验**：
   - 打开 `/Applications/<App>.app`，并通过 `pgrep -fl "<App>"` 确认进程正常存活。

## 4. 开源发布与 GitHub GEO 优化清单

开源或打磨 GitHub 仓库时，按以下清单完善：

- **仓库元数据**：通过 `gh repo edit` 设置高信息密度关键词简介（`description`）和 10 个以上精准 `topics`。
- **GEO 结构化 `README.md`**：
  - 顶部放置状态徽章、`English | [简体中文](README.zh-CN.md)` 切换链接与软件截图（`screenshots/`）。
  - 开头给出清晰的 **TL;DR 定义段**（直接回答项目解决了什么问题）。
  - 包含**横向对比表格**、**完整支持型号清单**、自然语言 **FAQ 问答区**。
  - 提供 **GitHub Releases** 预编译包下载步骤，并附上首次打开绕过 Gatekeeper 的命令：`xattr -cr "/Applications/<App>.app"`。
- **Release 与推送**：
  - 上传压缩后的预编译 `.app`（`*-arm64.zip`）并撰写结构化中英双语 Release Notes。
  - 若 `git push` 在 HTTP/2 下卡住，改用 HTTP/1.1 推送：
    ```bash
    git -c http.postBuffer=524288000 -c http.version=HTTP/1.1 push origin <branch>
    ```
