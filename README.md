# dev-workflow-preferences

> **English** | [简体中文](README.zh-CN.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

An AI coding agent **Skill** (`SKILL.md`) capturing personal development, UI/UX, `.NET` macOS `arm64` build/signing, and GitHub open-source/GEO workflows—so every new workspace or machine follows the same standards out of the box.

## What This Skill Covers

1. **Communication & Execution Style**: Simplified Chinese replies without emojis, proposing plans before broad/open-ended changes, and reusing proven UI patterns.
2. **Product, UI & Localization**: Native AppKit Liquid Glass desktop widgets, minimalist 3-item tray menus, industrial-style local web setup workbenches, real-time bar + spline interactive charts, and English-by-default multi-language support (`en` + `zh-CN`).
3. **macOS `.NET` Build & Ad-Hoc Signing**: Single-arch `osx-arm64` publishing, preserving `AppIcon.icns`, cleaning duplicate intermediate `.app` bundles, signing `MonoBundle/*.dylib` individually before the `.app`, and verifying runtime PIDs.
4. **Open-Source & GitHub GEO Checklist**: Keyword-rich repository descriptions & topics, GEO-structured `README.md` (TL;DR, comparison tables, FAQs, `xattr -cr` Gatekeeper hints), bilingual docs, and reliable `HTTP/1.1` git pushing.

## Quick Install (Global Skill)

Clone directly into your Antigravity global skills directory (`~/.gemini/config/skills/`):

```bash
mkdir -p ~/.gemini/config/skills
git clone https://github.com/ARRIIIIIS/dev-workflow-preferences.git ~/.gemini/config/skills/dev-workflow-preferences
```

To update later on any machine:

```bash
git -C ~/.gemini/config/skills/dev-workflow-preferences pull
```

## License

MIT
