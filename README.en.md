# resume-md2pdf

English · **[简体中文](README.md)**

Turn a **Markdown résumé** into a polished, role-tailored PDF: apply an opinionated visual theme, preview it as HTML, then export an **A4 vector PDF** with **Puppeteer / real Chromium** (selectable & copyable text, crisp at any zoom; pagination controlled precisely via `@page` / `break-inside` so lines are never cut mid-row).

At its core are three independent Node scripts (`build_html` / `check_theme` / `render_pdf`) that run from the command line — tool- and platform-agnostic. The only step that involves an AI assistant is *designing a visual theme for this résumé on the spot*; you can let an AI coding agent (e.g. Claude Code) do it, or hand-write a theme CSS yourself.

## Scope

- **Does**: Markdown layout + visual theme + HTML preview + A4 vector PDF export.
- **Does not**: rewrite / polish / add / remove résumé content (content optimization is a separate concern); no mixed CJK/Latin typesetting (Chinese by default, or English only).

## Workflow

1. **Identify the target role** (user-stated > inferred from the résumé > generic default), confirm language.
2. **Pick a theme style** (asked before generating HTML):
   - **Brand design style** — based on the 67 brands in [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) (Stripe / Apple / Linear / Notion / Claude …): `curl` that brand's `DESIGN.md`, distill its palette/typography/mood into a résumé theme. See `references/brand-styles.md`.
   - **Freeform design** — let an AI coding agent (e.g. Claude Code's `frontend-design` skill) design one on the spot, or hand-write a theme CSS yourself against the constraints. See `references/theme-design-guide.md`.
3. **Validate the theme**: `node scripts/check_theme.mjs .resume-theme.css` (blocks pagination overrides / missing `--accent` / image backgrounds, etc.).
4. **Generate HTML preview** (opens in browser): `node scripts/build_html.mjs --in resume.md --theme-css .resume-theme.css --lang zh-CN [--icons]`.
5. **Export PDF after the user approves**: `node scripts/render_pdf.mjs --in resume.html --out resume.pdf`.

## Two hard guarantees

- **No mid-line cuts**: pagination is delegated to Puppeteer's native `@page` + `break-inside`/`break-after` + `widows/orphans` (defined in `templates/base.css`); themes must not override them.
- **Vector, selectable & copyable, zoom-faithful**: the PDF is produced by `page.pdf()`; screenshot rasterization is forbidden.

## Install

```bash
cd scripts && npm install      # installs marked + puppeteer (first run downloads Chromium, a few hundred MB)
```

Chinese fonts / troubleshooting: see `references/setup-notes.md`.

## Usage

The three scripts are independent and can be called on their own. **When `--out` is omitted, HTML / PDF default to the Desktop** (macOS `~/Desktop`, Windows equivalent; auto-falls back to `OneDrive\Desktop` if OneDrive redirection is on).

`build_html`'s default output and `render_pdf`'s default input are both `~/Desktop/resume.html`, so the two chain together without passing any path.

### Generate an HTML preview

```bash
node scripts/build_html.mjs --in resume.md --theme-css .resume-theme.css --lang zh-CN --icons
```

| Flag | Required | Description | Default |
|---|---|---|---|
| `--in` | yes | Markdown résumé | — |
| `--theme-css` | yes | Theme CSS (AI-designed or hand-written) | — |
| `--out` |  | Output HTML path | `~/Desktop/resume.html` |
| `--lang` |  | Language (`zh-CN` / `en`) | `zh-CN` |
| `--title` |  | HTML `<title>` | by language (个人简历 / Resume) |
| `--theme-name` |  | Theme name (body class + annotation) | `custom` |
| `--icons` |  | Add Lucide icons to contact info | off |
| `--no-open` |  | Don't auto-open the browser preview | auto-open by default |

<details><summary>Flag details</summary>

- `--in`: the Markdown résumé source — the only content input.
- `--theme-css`: the visual skin, layered on `templates/base.css` (layout / pagination); decides only "how it looks". AI-designed or hand-written, but must pass `check_theme` first.
- `--out`: output HTML path; defaults to the Desktop.
- `--lang`: written to `<html lang>`, picks the CJK / Latin font stack and affects the `--title` default.
- `--title`: the browser tab / bookmark name.
- `--theme-name`: theme identifier — goes into `<body class="theme-<name>">`, a comment atop the inlined CSS, and the console log; theme CSS can use this class as a named scope (e.g. `.theme-stripe h2`). Defaults to `custom`.
- `--icons`: only matches icons for the contact line; does not affect body text or layout.
- `--no-open`: skip auto-opening the browser preview — handy for batch runs or being called by another script.

</details>

> Phone numbers in the contact line have dashes stripped automatically (`139-8765-4321` → `13987654321`); no need for `--icons`.

### Validate the theme (must pass before generating HTML)

```bash
node scripts/check_theme.mjs .resume-theme.css   # non-zero exit = violation; fix per the message and re-check
```

> The argument is the theme CSS **path** (a positional arg, not a `--flag`); exit code 0 = pass, non-zero = violation (missing `--accent`, `@page` present, pagination overridden, image background, etc.) — fix per the printed message and re-check.

### Export the PDF

```bash
node scripts/render_pdf.mjs --in resume.html --out resume.pdf
```

| Flag | Required | Description | Default |
|---|---|---|---|
| `--in` |  | Input HTML | `~/Desktop/resume.html` |
| `--out` |  | Output PDF | `~/Desktop/resume.pdf` |
| `--no-open` |  | Don't auto-open the PDF preview | auto-open with the system default app |

> `--in` / `--out` default to align with `build_html` (Desktop `resume.html` ↔ `resume.pdf`), so the two chain together without passing any path; `--no-open` disables auto-preview, otherwise the PDF opens with the system default app (Preview on macOS, often Edge on Windows).

## Layout

```
SKILL.md                      Entry point (trigger description + workflow + asset map)
agents/interface.yaml         Interface metadata
scripts/build_html.mjs        Markdown → themed HTML (semantic grouping + optional Lucide icons + auto-open preview)
scripts/check_theme.mjs       Theme-CSS hard-constraint validator (no deps)
scripts/render_pdf.mjs        HTML → A4 vector PDF (Puppeteer, preferCSSPageSize)
templates/base.css            Structure layer: A4 / font stack / pagination rules (stable base)
templates/resume.template.html HTML skeleton
assets/lucide-icons.js        Inline Lucide SVG icon set
references/theme-design-guide.md  How to invoke frontend-design for a theme (prompt template + CSS contract)
references/brand-styles.md        Brand design style library (67 brands + how to fetch & adapt)
references/style-examples/        Reference CSS for inspiration (borrow patterns, don't copy)
references/setup-notes.md          Setup & troubleshooting
references/export-checklist.md     Export checklist
examples/sample-resume.md          A ready-to-run sample résumé
```

## License

[MIT](LICENSE). Brand design-spec material is sourced from the third-party repository [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md); its content remains the property of the respective owners.
