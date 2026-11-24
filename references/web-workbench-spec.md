# 工业风 Web 设置向导规范

适用于嵌入桌面端本地 HTTP 服务（如 `HttpListener`）的浏览器配置工作台，参考 Fitbit 与 Galaxy Buds 项目实践。

## 1. 视觉设计系统（CSS Tokens）

采用 **深色工业风切角美学（Dark Industrial Chamfer）**：

```css
:root {
  --bg: #060609;
  --surface: rgba(13, 14, 19, 0.88);
  --surface-solid: #0d0e13;
  --border: rgba(255, 255, 255, 0.12);
  --border-hi: rgba(255, 255, 255, 0.28);
  --grid-line: rgba(255, 255, 255, 0.065);
  --well: rgba(255, 255, 255, 0.03);
  --field: #090a0e;
  --text: #ffffff;
  --text-2: rgba(255, 255, 255, 0.68);
  --text-3: rgba(255, 255, 255, 0.42);
  --neon: #c1ff12;
  --neon-soft: rgba(193, 255, 18, 0.12);
  --cta-fg: #060609;
  --font: -apple-system, BlinkMacSystemFont, "SF Pro Display", "PingFang SC", sans-serif;
  --mono: "SF Mono", ui-monospace, Menlo, monospace;
  --chamfer: polygon(10px 0, 100% 0, 100% calc(100% - 10px), calc(100% - 10px) 100%, 0 100%, 0 10px);
  --chamfer-sm: polygon(6px 0, 100% 0, 100% calc(100% - 6px), calc(100% - 6px) 100%, 0 100%, 0 6px);
}
```

- **背景层次**：`#060609` 底色 + 固定全屏 `72px × 72px` 细网格线 + 顶部椭圆形低饱和紫/粉/荧光绿径向光晕（径向遮罩衰减）。
- **斜切角造型**：主容器、卡片、主操作按钮使用 `clip-path: var(--chamfer)` 或 `--chamfer-sm` 替代常规 `border-radius`，强化硬朗工程仪表感。
- **字体排印**：正文使用系统无衬线字体；数字、状态标签、频段刻度、百分比与版本号统一使用 `var(--mono)` + `font-variant-numeric: tabular-nums`。

## 2. 页面信息架构与交互原则

- **居中单栏工作台**：`.container { width: 100%; max-width: 660px; }`，左右两侧保留 `1px solid var(--grid-line)` 工程边界线。
- **顶部区域**：
  1. Hero 徽章 + 主标题 + 副标题；右上角固定 `EN | 中文` 双语切换胶囊。
  2. **HUD 遥测状态条**：4 格核心读数（如 `STATUS`、`L BUD`、`R BUD`、`CASE`），实时轮询 `/api/state` 更新。
- **分步式 Stepper 导航**：
  - 将功能拆分为 `01`、`02`、`03` 三个清晰步骤（如蓝牙与型号 / 音频与双设备 / 坐姿与组件）。
  - 模块小标题统一采用 `<b>01</b> // SECTION TITLE` 工业编号风格。
- **所见即所得与即时生效**：
  - 选型下拉框（型号 + 配色）联动实时切换左右耳 **3D/产品实物渲染图**。
  - 开关（Switch）、模式按钮、滑块等调节操作**即时发送 `POST /api/*` 生效，不设多余“保存”按钮**；仅“写入配置”或“一键校准”等一次性指令配明确触发按钮与短暂 Toast 反馈。

## 3. 音频/传感器曲线（EQ 交互规范）

- **柱状图 + 平滑曲线双层联动**：
  - 底层为 9 段垂直柱状图（`63 / 125 / 250 / 500 / 1k / 2k / 4k / 8k / 16k`，范围 `-10 dB` ~ `+10 dB`，以 `0 dB` 中线为基准向上/向下延伸）。
  - 顶层叠加 SVG 三次贝塞尔平滑曲线（`#c1ff12`）与半透明面积填充及可拖动圆点。
- **实时响应**：指针按住拖动时实时重绘柱高与曲线路径（`drawEqChart`），松开指针（`pointerup`）立即下发自定义参数至设备；选择官方预设时曲线同步切换预览。

## 4. 前后端语言联动

- 页面元素标注 `data-i18n` 属性，内置 `I18N.en`（默认）与 `I18N.zh` 字典。
- 点击右上角语言胶囊调用 `POST /api/lang`，后端更新持久化配置 `Locale`，同步刷新 Web 页面、原生桌面小组件与菜单栏托盘。
