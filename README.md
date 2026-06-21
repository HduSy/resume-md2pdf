# resume-md2pdf

**[English](README.en.md)** · 简体中文

把 **Markdown 简历**按目标岗位套上有设计观点的视觉主题，先渲染成 HTML 预览，确认后用 **Puppeteer / 真实 Chromium** 导出 **A4 矢量 PDF**（文字可选可复制、缩放保真；分页用 `@page` / `break-inside` 精确控制，不会从一行中间截断）。

核心是三个互相独立的 Node 脚本（`build_html` / `check_theme` / `render_pdf`），纯命令行即可跑通，不绑定任何特定工具或平台。唯一用到 AI 助手的地方是「为这份简历现场设计一套视觉主题」——这一步既可以交给 AI 编码助手（如 Claude Code），也可以自己手写一份主题 CSS。

## 能力边界

- **做**：Markdown 排版 + 视觉主题 + HTML 预览 + 导出 A4 矢量 PDF。
- **不做**：不改写 / 不润色 / 不增删简历内容（内容优化是另一回事）；不做中英混排（默认中文，或纯英文）。

## 工作流

1. **定位目标岗位**（用户明说 > 从简历推断 > 默认通用），确认语言。
2. **选主题风格**（生成 HTML 前先问）：
   - **品牌设计风格** —— 基于 [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) 的 67 个知名品牌（Stripe / Apple / Linear / Notion / Claude …），`curl` 取该品牌 `DESIGN.md`，提炼配色/字体/气质转成简历主题。见 `references/brand-styles.md`。
   - **自由设计** —— 让 AI 编码助手（如 Claude Code 的 `frontend-design` skill）现场设计，或自己按约束手写一份主题 CSS。见 `references/theme-design-guide.md`。
3. **校验主题**：`node scripts/check_theme.mjs .resume-theme.css`（拦截改分页 / 缺 `--accent` / 图片背景等违规）。
4. **生成 HTML 预览**（自动开浏览器）：`node scripts/build_html.mjs --in resume.md --theme-css .resume-theme.css --lang zh-CN [--icons]`。
5. **用户确认后导出 PDF**：`node scripts/render_pdf.mjs --in resume.html --out resume.pdf`。

## 两条硬保证

- **不从一行中间截断**：分页交给 Puppeteer 原生 `@page` + `break-inside`/`break-after` + `widows/orphans`（写在 `templates/base.css`），主题不得覆盖。
- **矢量、可选可复制、缩放保真**：PDF 由 `page.pdf()` 产出，禁止截图栅格化。

## 安装

```bash
cd scripts && npm install      # 安装 marked + puppeteer（首次会下载 Chromium，约几百 MB）
```

中文字体 / 排错见 `references/setup-notes.md`。

## 用法

三个脚本互相独立，可单独调用。**未传 `--out` 时，HTML / PDF 默认输出到桌面**（macOS `~/Desktop`、Windows 同理；开了 OneDrive 重定向时自动回退 `OneDrive\Desktop`）。

`build_html` 的默认输出与 `render_pdf` 的默认输入同为桌面 `resume.html`，二者不传路径即可首尾相连。

### 生成 HTML 预览

```bash
node scripts/build_html.mjs --in resume.md --theme-css .resume-theme.css --lang zh-CN --icons
```

| 参数 | 必填 | 说明 | 默认 |
|---|---|---|---|
| `--in` | 是 | Markdown 简历 | — |
| `--theme-css` | 是 | 主题 CSS（AI 现场设计或手写） | — |
| `--out` |  | 输出 HTML 路径 | `~/Desktop/resume.html` |
| `--lang` |  | 语言（`zh-CN` / `en`） | `zh-CN` |
| `--title` |  | HTML `<title>` | 按语言（个人简历 / Resume） |
| `--theme-name` |  | 主题名（body class + 注释标识） | `custom` |
| `--icons` |  | 给联系方式加 Lucide 图标 | 默认关 |
| `--no-open` |  | 不自动开浏览器预览 | 默认自动开 |

<details><summary>参数说明</summary>

- `--in`：简历的 Markdown 源文件，唯一的内容输入。
- `--theme-css`：视觉皮肤，叠加在 `templates/base.css`（管排版 / 分页）之上，只决定「长什么样」；AI 现场设计或手写均可，须先过 `check_theme`。
- `--out`：HTML 输出路径，不传则落桌面。
- `--lang`：写进 `<html lang>`，决定中 / 英字体栈，也影响 `--title` 的默认值。
- `--title`：浏览器标签页 / 书签名。
- `--theme-name`：主题标识，同时进入 `<body class="theme-<名字>">`、内联 CSS 顶部注释、控制台日志；主题 CSS 可借这个 class 写命名作用域（如 `.theme-stripe h2`）。不传为 `custom`。
- `--icons`：仅给联系方式行匹配图标，不影响正文与排版。
- `--no-open`：不自动开浏览器预览，批量生成或被其它脚本调用时常用。

</details>

> 联系方式行的手机号会自动去掉连字符（`139-8765-4321` → `13987654321`），无需 `--icons`。

### 校验主题（生成 HTML 前必过）

```bash
node scripts/check_theme.mjs .resume-theme.css   # 非 0 退出即违规，按报错修正后重校
```

> 参数是主题 CSS 的**路径**（位置参数，不是 `--flag`）；退出码 0 = 通过，非 0 = 违规（缺 `--accent` / 写了 `@page` / 覆盖分页 / 图片背景等），按打印的提示修正后重校。

### 导出 PDF

```bash
node scripts/render_pdf.mjs --in resume.html --out resume.pdf
```

| 参数 | 必填 | 说明 | 默认 |
|---|---|---|---|
| `--in` |  | 输入 HTML | `~/Desktop/resume.html` |
| `--out` |  | 输出 PDF | `~/Desktop/resume.pdf` |
| `--no-open` |  | 不自动打开 PDF 预览 | 默认用系统默认应用打开 |

> `--in` / `--out` 默认与 `build_html` 对齐（桌面 `resume.html` ↔ `resume.pdf`），二者都不传路径即可首尾相连；`--no-open` 关掉自动预览，否则 PDF 用系统默认应用打开（macOS 为 Preview、Windows 常为 Edge）。

## 目录

```
SKILL.md                      入口（触发描述 + 工作流 + 资产地图）
agents/interface.yaml         接口元数据
scripts/build_html.mjs        Markdown → 主题化 HTML（结构分组 + 可选 Lucide 图标 + 自动开预览）
scripts/check_theme.mjs       主题 CSS 硬约束校验（无依赖）
scripts/render_pdf.mjs        HTML → A4 矢量 PDF（Puppeteer，preferCSSPageSize）
templates/base.css            结构层：A4 / 字体栈 / 分页规则（稳定地基）
templates/resume.template.html HTML 骨架
assets/lucide-icons.js        内联 Lucide SVG 图标集
references/theme-design-guide.md  如何调用 frontend-design 生成主题（含 prompt 模板与 CSS 契约）
references/brand-styles.md        品牌设计风格库（67 个品牌 + 取用与转化）
references/style-examples/        设计灵感范例 CSS（借模式，不照搬）
references/setup-notes.md          配置与排错
references/export-checklist.md     导出检查清单
examples/sample-resume.md          可直接试跑的样例简历
```

## 许可

[MIT](LICENSE)。品牌设计规范素材来自第三方仓库 [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md)，其内容版权归各自所有者。
