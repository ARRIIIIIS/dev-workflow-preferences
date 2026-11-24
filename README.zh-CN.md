# dev-workflow-preferences

> [English](README.md) | **简体中文**

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

用于 AI 编程助手的个人开发习惯 **Skill（`SKILL.md`）**，沉淀了 macOS 桌面组件 UI/UX、`.NET` `arm64` 打包与 ad-hoc 签名防坑流程，以及 GitHub 开源与 GEO 优化规范，方便在不同电脑和项目间一键复用与持续迭代。

## 包含规范

1. **沟通与执行风格**：简体中文、无 emoji、开放性需求先给方案确认再动手，明确任务直接执行到底；界面风格优先对齐既有项目。
2. **产品、UI 与多语言偏好**：原生 AppKit Liquid Glass 桌面小组件 + 极简 3 项菜单栏 + 工业风网页工作台；柱状图与平滑曲线实时联动；界面与文档默认英文，支持简体中文实时切换。
3. **macOS `.NET` 构建与签名规范**：单架构 `osx-arm64` 发布、保留 `AppIcon.icns`、清理中间 `.app` 防止重复、先逐个签名 `MonoBundle/*.dylib` 再签名 `.app`，并校验进程启动。
4. **开源发布与 GitHub GEO 清单**：完善仓库 `description` 与 `topics`、GEO 结构化 `README.md`（TL;DR、对比表、支持清单、FAQ、`xattr -cr` 解锁提示）、中英双语文档与 `HTTP/1.1` 稳定推送。

## 一键安装（全局 Skill）

克隆到 Antigravity 全局 Skills 目录（`~/.gemini/config/skills/`）即可在所有项目中生效：

```bash
mkdir -p ~/.gemini/config/skills
git clone https://github.com/ARRIIIIIS/dev-workflow-preferences.git ~/.gemini/config/skills/dev-workflow-preferences
```

后续在任意设备同步更新：

```bash
git -C ~/.gemini/config/skills/dev-workflow-preferences pull
```

## 开源协议

MIT
