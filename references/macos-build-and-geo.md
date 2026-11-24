# macOS .NET 发布打包、签名与 GitHub GEO 规范

## 1. .NET 10 macOS (`arm64`) 编译与安装流程

1. **原生动态库编译**（如有 Objective-C/C++ 组件）：先执行原生构建脚本（如 `build-buds-widget.sh`）生成 `lib*.dylib`。
2. **强制单架构发布 `osx-arm64`**：
   - 使用 `$HOME/.dotnet-sdk10`（`DOTNET_ROOT="$HOME/.dotnet-sdk10"`）。
   - **禁止使用 Universal (`x86_64;arm64`) 发布**：在 macOS 26/27+ 上 Universal 构建容易触发 `SIGKILL (Code Signature Invalid)`。
   ```bash
   DOTNET_ROOT="$HOME/.dotnet-sdk10" PATH="$HOME/.dotnet-sdk10:$PATH" \
   dotnet publish <Project.csproj> -c Release -r osx-arm64 \
     -p:UseMacOSSDK=true \
     -p:AllowMissingPrunePackageData=true \
     -p:ValidateXcodeVersion=false
   ```
3. **安装到 `/Applications` 与图标注入**：
   - 使用 `ditto` 将生成的 `.app` 复制到 `/Applications/<App>.app`。
   - 将 `AppIcon.icns` 复制到 `Contents/Resources/AppIcon.icns`，并确保 `Info.plist` 指向它：
   ```bash
   plutil -replace CFBundleIconFile -string "AppIcon.icns" "/Applications/<App>.app/Contents/Info.plist"
   ```
4. **清理中间 `.app` 防止重复**：
   - 删除 `bin/` 和 `obj/` 下生成的中间 `.app` 包，避免 Spotlight 和 Launchpad 出现重复应用入口：
   ```bash
   find <repo>/bin/Release <repo>/obj/Release -name "*.app" -type d -exec rm -rf {} + 2>/dev/null || true
   ```
5. **逐个 Ad-Hoc 签名（避坑关键）**：
   - `codesign --deep` 会跳过 `Contents/MonoBundle/*.dylib`，必须先逐个签完所有 `.dylib`，再签 `.app`：
   ```bash
   for f in "/Applications/<App>.app/Contents/MonoBundle/"*.dylib; do
     codesign --force --sign - "$f" >/dev/null 2>&1
   done
   codesign --force --sign - "/Applications/<App>.app"
   ```
6. **启动与进程校验**：
   ```bash
   pkill -f "<App>" || true
   open "/Applications/<App>.app"
   sleep 2
   pgrep -fl "<App>"
   ```

## 2. GitHub 开源与 GEO（生成式引擎优化）清单

目标是让 ChatGPT、Perplexity、Claude、Google AI Overview 与 GitHub Search 能高概率检索并引用项目：

1. **仓库元数据（`gh repo edit`）**：
   - `description`：一句高信息密度英文描述，覆盖平台、形态与核心卖点。
   - `topics`：至少配置 10 个核心检索词（如 `macos`、`desktop-widget`、`liquid-glass`、`apple-silicon`、`dotnet`、`appkit`、`menubar-app` 及对应硬件/领域词）。
2. **`README.md`（英文主文档）+ `README.zh-CN.md`**：
   - 顶部放状态徽章（Platform / Release / License）+ `English | 简体中文` 切换 + 软件截图（`screenshots/`）。
   - **TL;DR 定义段**：开篇直接陈述“痛点 → 解决方案”（如官方无 Mac 版，本项目如何原生解决），方便 AI 直接摘录。
   - **横向对比表**：用 Markdown 表格对比官方/传统客户端与本项目的差异（AI 引擎极偏好引用结构化表格）。
   - **支持型号/硬件全称清单**：列出完整型号名称以覆盖按型号检索的长尾流量。
   - **FAQ 问答区**：用自然语言提问作小标题（如 *Is there an official ... for macOS?*、*How to fix unidentified developer?*）。
   - **安装指引**：写明 GitHub Releases 下载与 `xattr -cr "/Applications/<App>.app"` 首次打开解锁命令。
3. **GitHub Release**：
   - 上传打包好的 `*-arm64.zip`（如 `ditto -c -k --keepParent "/Applications/<App>.app" "<Name>-arm64.zip"`）。
   - 撰写结构化中英双语 Release Notes（Highlights + Installation + 中文摘要）。
4. **网络稳定性推送**：
   - 若遇到 `git push` 在 HTTP/2 下挂起，使用 HTTP/1.1 与大缓冲区推送：
   ```bash
   git -c http.postBuffer=524288000 -c http.version=HTTP/1.1 push origin <branch>
   ```
