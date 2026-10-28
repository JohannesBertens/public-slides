# Headless slide creation options — October 2026 survey

Research date: **2026-10-28**. Scope: options for creating presentation decks *without a GUI authoring app* — interactive server/client presentation frameworks on one axis, and PowerPoint (PPTX) export on the other. Motivation for this repo: the current deck is a ~165 MB binary PPTX stored in Git LFS; text-first authoring with generated exports would shrink the repo and make decks diff-able.

Method: every version/activity claim below was checked against primary sources on 2026-10-28 (GitHub releases & repo push dates, npm/PyPI registry data, official docs). Fidelity claims come from the tools' own documentation. AI slide SaaS (Gamma, Beautiful.ai, …) is out of scope — none of them offer a headless/programmatic authoring model that fits a git-based repo.

## TL;DR

- **Interactive server/client**: [Slidev](https://sli.dev/) (v53) and [Marp CLI](https://github.com/marp-team/marp-cli) (v4.5) are the two healthy, actively-shipped markdown-deck ecosystems. Slidev has the richest presenter story (dev server, presenter mode, `--remote` with password-protected remote control, static SPA build). Marp has the simplest server story (`marp -s ./slides` serves on-demand HTML/PDF/PPTX/PNG over HTTP). Quarto wraps reveal.js with a scientific-authoring workflow; Presenterm presents markdown straight in the terminal.
- **PPTX export, two very different animals** (verify which one your recipients need):
  - *Image-wrapped* (each slide = one picture; visually faithful, not editable, text not selectable): Slidev `--format pptx`, Marp `--pptx`. Both carry presenter notes.
  - *Native-editable* (real text boxes/shapes): Slidev's new `--format pptx-editable` (hybrid: native shapes, browser-only effects stay per-element pictures), Marp's experimental `--pptx-editable` (LibreOffice-mediated, rougher), and Pandoc/Quarto's pptx writer (fully native, but layout-grammar-constrained). For full programmatic control: PptxGenJS / python-pptx / Open XML SDK (all MIT) or Aspose.Slides (commercial, most complete engine).

---

## Track A — interactive server/client presentation frameworks

| Tool | Version (released) | Activity | License | Authoring | Server/client interactivity | PPTX story |
|---|---|---|---|---|---|---|
| [Slidev](https://sli.dev/) | 53.0.0 (2026-09-16) | pushed 2026-10-23, ~49k★ | MIT | Markdown + Vue/Vite, live components, code highlighting, Mermaid, KaTeX | dev server w/ live reload; presenter mode; `slidev --remote [pw]` = public host + remote control, optional password; `slidev build` → static SPA (`--base`, `--without-notes`) | `slidev export --format pptx` (image slides) and `--format pptx-editable` (native shapes) |
| [Marp CLI](https://github.com/marp-team/marp-cli) | 4.5.1 (2026-09-06) | pushed 2026-09-08, ~3.9k★ (ecosystem ~12.7k★) | MIT | Markdown, directives, themes, math, images | watch mode; **server mode** `marp -s ./slides`: on-demand HTTP conversion (`deck.md?pdf`, `?pptx`, `?png`, …) | `--pptx` (image slides) and experimental `--pptx-editable` (LibreOffice-based) |
| [reveal.js](https://revealjs.com/) | 6.0.2 (2026-09-10) | pushed 2026-09-30, ~72k★ | MIT | HTML + markdown plugin | any web server; print-to-PDF; speaker notes plugin | none native (screenshot pipelines only) |
| [Quarto](https://quarto.org/) | 1.10.19 (2026-10-06) | pushed 2026-10-27, ~6.1k★ | GPL-2+/LGPL/MIT mix | `.qmd` markdown; executable code cells (Jupyter/knitr/OJS); `revealjs` and `pptx` and `beamer` formats from one source | Quarto preview server; reveal.js output hosts anywhere | `format: pptx` = pandoc writer (native, constrained) |
| [Presenterm](https://mfontanini.github.io/presenterm/) | 0.16.1 (2026-02-20) | pushed 2026-10-28, ~8.9k★ | MIT | Markdown in your editor, presented in the terminal (kitty/iTerm2/wezterm/ghostty/foot: images & animated GIFs) | hot reload on save; snippet execution inside pseudo-TTYs; speaker notes | export to PDF and HTML only — no PPTX |
| [Spectacle](https://github.com/FormidableLabs/spectacle) | 10.2.4 (2026-10-27) | pushed 2026-10-27, ~10.2k★ | MIT | React/TypeScript components | browser presentation, framework-native | none built in (it's React → your own export pipeline) |
| mdx-deck | — | last push 2023-01 — **unmaintained** | MIT | MDX | — | — |

Notes:

- **Slidev interactivity caveat** (from the export docs): interactive features are *not* available in exported files — to keep them, build and host the deck as a web app (`slidev build`). That makes Slidev the strongest fit when the deliverable is a URL, and the exports are for offline/presenter fallback.
- **Marp server mode** is the only one of the two that serves *conversions on demand over HTTP* (`http://localhost:8080/deck-a.md?pdf` returns a converted PDF) — handy as a tiny deck-hosting service for a folder of talks. PDF/PPTX/PNG conversion requires a local Chrome/Edge/Firefox; the official `marpteam/marp-cli` Docker image bundles everything.
- **reveal.js 6** dropped down to a lean plugin set (markdown, highlight, math, notes, search, zoom). The old multiplex plugin is no longer bundled — remote-audience sync is now DIY/SocketIO territory. For HTML→PDF of web decks, [Decktape](https://github.com/astefanutti/decktape) (3.17.0, 2026-10-19) remains the standard headless renderer.
- **Quarto** explicitly positions `revealjs` as its most capable presentation format; the `pptx` format is there when recipients require Office files. One `.qmd` source → reveal.js deck *and* PPTX *and* Beamer PDF is its unique trick.

## Track B — PPTX export paths (the fidelity question)

The single most important question when someone asks for "a PPTX": **do recipients need to edit it, or just present/open it?**

| Path | Mechanism | Fidelity class | Presenter notes | Editable by recipient | Caveats |
|---|---|---|---|---|---|
| Slidev `slidev export --format pptx` | Playwright screenshots → one image per slide | **image-wrapped** | ✅ carried per-slide | ❌ (text not selectable) | `--with-clicks` on by default (one slide per click step) |
| Slidev `slidev export --format pptx-editable` | slides measured in browser → rebuilt as native PowerPoint shapes | **native-editable (hybrid)** | ✅ | ✅ text/shapes; SVG, canvas, iframe, video, KaTeX, CSS gradients/filters, `clip-path` stay *pictures* (per element, not per slide) | unsafe slides fall back to image export; fonts are named, not embedded; text wrap can differ from browser; `::before/::after` decorations dropped |
| Marp `marp --pptx` | pre-rendered slide backgrounds | **image-wrapped** | ✅ | ❌ "contents cannot be modified or re-used" | opens fine in PowerPoint/Keynote/Google Slides/Impress |
| Marp `--pptx --pptx-editable` | browser + LibreOffice Impress conversion | **native-editable (experimental)** | ❌ not supported | ✅ | needs LibreOffice installed; can throw/incomplete output on complex themes (e.g. `gaia`); officially "lower slide reproducibility" |
| Pandoc `pandoc -t pptx` (3.13, 2026-10-26) | own pptx writer: content mapped into named layouts of a `reference.pptx` template | **native-editable** | ✅ | ✅ real text | fixed layout grammar (Title Slide / Two Content / Comparison / Content with Caption / Section Header / Blank / Title and Content); no arbitrary positioning; template must contain layouts with those names |
| Quarto `format: pptx` | same pandoc writer + template handling (`reference-doc`), `.columns` divs, incremental, background images | **native-editable** | ✅ | ✅ | same pandoc constraints; Quarto docs themselves say revealjs is the more capable format |
| [PptxGenJS](https://gitbrent.github.io/PptxGenJS/) (4.0.1, 2025-06) | JS/TS library builds OOXML directly; browser + Node | **native-editable** | ✅ | ✅ text, tables, shapes, charts, slide masters | last release mid-2025 (slowing, not abandoned); you write code, not markdown |
| [python-pptx](https://python-pptx.readthedocs.io/) (1.0.2, 2024-08, MIT) | Python library builds/edits OOXML | **native-editable** | ✅ | ✅ | stable but slow-moving (last release Aug 2024); no rendering — combine with LibreOffice for previews |
| [Open XML SDK](https://github.com/dotnet/open-xml-sdk) (3.5.1, 2026-03, MIT) / [ShapeCrawler](https://github.com/ShapeCrawler/ShapeCrawler) | .NET direct OOXML manipulation (ShapeCrawler = friendlier wrapper, active 2026-10) | **native-editable** | ✅ | ✅ | lowest-level of the programmatic options |
| [Aspose.Slides](https://products.aspose.com/slides/family/) (Python/.NET/Java/…; Python pkg 26.10.0, 2026-10-09) | commercial full presentation engine, headless (no PowerPoint needed) | **native-editable** + faithful conversions | ✅ | ✅ | paid license (evaluation watermark on free tier); can also *import* PDF pages and HTML content into slides, export PDF/TIFF/XPS/SWF/HTML/SVG/images, handle macros/math/encryption; monthly releases |
| LibreOffice headless (26.8 current) | `soffice --headless --convert-to pptx` | format **conversion** (e.g. ODP→PPTX), quality varies | usually | partially | it's the engine behind Marp's editable export and most CI pipelines; not an authoring tool |
| Decktape (3.17.0, 2026-10-19) | screenshots HTML decks to PDF | image-flattened PDF | notes pages separately | ❌ | PDF, not PPTX; pairs with third-party PDF→PPTX steps (image-wrapped again) |

### Observations

1. **2026's big change**: editable PPTX from web-first tools became real. Slidev's `pptx-editable` (browser-measured → native shapes, per-element image fallbacks) and Marp's LibreOffice-based `--pptx-editable` both exist now; a year ago the honest answer was "image slides only". Both are documented with explicit caveats — treat editable export as a first-class but imperfect path.
2. **Fonts are the silent killer of cross-machine fidelity** for all native-editable paths: PPTX names fonts without embedding them (Slidev's export even prints which families it referenced). Corporate templates' fonts must be installed on the machine that renders/converts.
3. **Image-wrapped is not a insult** for a talks repo: it is pixel-faithful, carries speaker notes, and opens everywhere. It only fails when someone must edit the deck afterwards.
4. **Pandoc/Quarto's writer is the only fully-native markdown→PPTX path** and it shows: great for content-driven decks (bullets, columns, images, tables), wrong for anything layout-expressive.

## Pipelines for a git-hosted talks repo

```text
deck.md ──slidev dev/build──▶ interactive SPA (GitHub Pages / Netlify)
deck.md ──slidev export──▶ slides.pdf  (presenter fallback)
deck.md ──slidev export --format pptx──▶ share.pptx (image-wrapped, notes included)
deck.md ──slidev export --format pptx-editable──▶ editable.pptx (native shapes)

slides/ ──marp -s──▶ https://host/deck.md        (HTML, live)
                   ▶ https://host/deck.md?pdf    (on demand)
                   ▶ https://host/deck.md?pptx   (on demand)
```

CI is unproblematic: Marp ships an official Docker image; Slidev needs `playwright-chromium`; the editable-Marp path additionally needs LibreOffice. [Gotenberg](https://gotenberg.dev/) (8.39.0, 2026-10-27, active) wraps LibreOffice conversions as a Docker HTTP API if you ever need office→PDF as a service.

## Recommendation matrix (for *this* repo)

| If the goal is… | Pick | Why |
|---|---|---|
| Best interactive web deck + presenter tooling, heavy code content | **Slidev** | richest dev/present/remote experience; SPA hosting keeps interactivity; pptx + pptx-editable exports for Office people |
| Minimal toolchain, deck folder served as a website with on-demand exports | **Marp CLI** | one npm package (or Docker image), server mode converts via URL query, watch mode, GitHub Pages for plain HTML |
| Scientific/technical decks, code output on slides, one source → web + office + PDF | **Quarto** | executable cells; revealjs + pptx + beamer from one `.qmd`; pptx is native but layout-constrained |
| Terminal-native presenting, no browser at all | **Presenterm** | PDF/HTML export for distribution; no PPTX |
| Recipients must *edit* the PPTX in PowerPoint, high visual fidelity | **Slidev `pptx-editable`** | native shapes with graceful per-element picture fallback; best editable fidelity of the 2026 field |
| Recipients must *edit* and content matters more than looks | **Pandoc/Quarto pptx** | fully native, template-driven, deterministic |
| Decks as code / programmatic generation, template enforcement | **python-pptx** or **PptxGenJS** (MIT), **Aspose** (commercial) | direct OOXML; Aspose adds faithful render/import (PDF, HTML) and a full engine |
| Just make the current 165 MB LFS deck smaller | move source to markdown, export artifacts in CI | text decks are KBs; binary PPTX stops living in git |

**Suggested default for this repo**: author in Slidev markdown (one `decks/<talk>/slides.md` per talk), host built SPAs on GitHub Pages, and let CI produce `pdf` + `pptx` artifacts per release — keeping Git LFS out of the talk pipeline entirely. Keep Marp as the lightweight fallback if Slidev's Node/Vite surface proves too heavy.
