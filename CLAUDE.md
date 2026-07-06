# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`hugo-azw-theme` is a custom Hugo theme for [blog.azw.jp](https://blog.azw.jp/). It is used as a git submodule from the parent blog repository at `../..` (the azw-blog root). The parent repo's CLAUDE.md has additional context about the overall blog setup.

**Design system**: All visual/styling work MUST follow `DESIGN.md` (Chromatic Neumorphism — design rules and the reasoning behind them). Read `DESIGN.md` in full BEFORE touching any template or SCSS — its §5 lists hard prohibitions (no third hue, no borders-for-depth, no backdrop-filter, no decoration inside article bodies) that are easy to violate accidentally. `DESIGN-BOOTSTRAP.md` holds migration history and the still-open items (share buttons, manual theme toggle); it is deleted once those are resolved.

The 2026-07 rebuild was validated against an approved mockup; `DESIGN.md` now carries all authoritative values (colors, shadows, component specs). If DESIGN.md and existing code disagree, treat it as a bug and ask — do not silently pick one.

## Development

```bash
# Run from the parent blog repo (../../), not from this theme directory
hugo server

# Production build
hugo --environment production
```

Hugo version: **0.147.5 extended** (extended edition required for SCSS compilation).

## Architecture

### Template Hierarchy

`baseof.html` is the root template. Since the 2026-07 rebuild, page layout uses CSS Grid (`.page-cols`, with `.has-toc` adding a 200px left TOC rail on article pages that have headings), not Bulma columns. Key structural pieces:

- `partials/header.html` — single-row flex header: brand (site title + `AZW.JP` sublabel) left, round nav buttons right. There is NO hero/header image anymore (`params.site.header` is ignored; do not re-add the banner).
- `partials/summary.html` — each post is an independent neumorphic card (`.article-summary`): 190px thumbnail grid, monospace date (`.meta-date`), title, excerpt, pill tags (`.tag-pills` / `.pill`).
- `partials/post/tags.html` — shared pill component (category pills = green, tag pills = pink).
- `partials/_widgets/*` — sidebar widgets are neu cards; list items are inset "wells" (`neu-well` mixin). `recent_posts.html` renders 2-line-clamped titles + dates, no thumbnails.

Key extension points for the parent site:
- `head.html` checks for `layouts/partials/myhead.html` in the parent repo — if it exists, it's injected at the end of `<head>`.
- `footer.html` checks for `layouts/partials/myfooter.html` in the parent repo — if it exists, it's injected after the footer.

### Styling

All custom CSS is in a single SCSS file: `assets/scss/style.scss`. It is compiled, minified, and fingerprinted by Hugo's asset pipeline (see `head.html` line 22).

**Dark mode** uses `@media (prefers-color-scheme: dark)` with CSS custom properties. All colors are exposed as CSS custom properties and re-wrapped into SCSS variables for use throughout the file.

**CRITICAL rule for light/dark differences**: mode differences live ONLY in the token definitions in `_variables.scss`. Component SCSS must never contain its own dark-mode `@media` branch. When a decoration should exist only in dark mode, define a token that is `none` in light and the real value in dark (see `--pagination-active-glow`, `--widget-title-glow`) and reference the token unconditionally. This keeps the planned `data-theme` manual toggle feasible.

**Neumorphism design**: depth comes from soft box-shadows via the `neu-*` mixins in `_mixins.scss` (`neu-card`, `neu-well`, `neu-glow`, plus the older raised/groove set). Shadows are background-derived and TINTED (light: pinkish `rgba(168,112,138,.30)`; dark: `rgba(9,11,16,.6)`) — do not "correct" them back to neutral gray. Attention/glow is a separate layer (`--glow-pink`/`--glow-green`); the chromatic aberration signature is `chromatic-split()` and is restricted to the site title (always) and post titles (hover only) per DESIGN.md §3.2. Always-on glow is budgeted: max 3 element kinds per viewport (DESIGN.md §3.5) — count before adding any.

**Responsive breakpoints** (SCSS mixins): `@include smaller` (<768px), `@include middle` (768–1216px), `@include larger` (≥768px), `@include wide` (≥1216px).

### External Dependencies (CDN)

Loaded in `head.html` — not bundled:
- Bulma 0.9.0 (CSS framework)
- FontAwesome 6.4.2
- jQuery 3.4.1
- Lightbox2 2.7.1 (image galleries)
- Tocbot 4.11.1 (table of contents)

**Known pitfalls with these dependencies** (both have bitten us before):
- `<main>` carries Bulma's `.content` class, so `.content ul { list-style: disc }` (specificity 0-1-1) beats a bare class selector. Any new list-based component inside main needs specificity ≥ 0-2-0 (pattern used: `.content .tag-pills, .tag-pills { ... }`) plus an explicit `li { list-style: none }` / `li::marker { content: "" }` reset. Grep for `override bulma` comments for prior art.
- CDN scripts are loaded with `defer`. Inline init code (e.g., `tocbot-init.html`) must run inside a `DOMContentLoaded` listener, or it executes before the library exists (`tocbot is not defined`).

### Configuration

See `exampleSite/config.yml` for all supported params. Key theme-specific params:
- `params.site.header` — header image path
- `params.site.deafult_thumbnail` — fallback thumbnail (note: the typo "deafult" is intentional/legacy)
- `params.widgets` — list of sidebar widgets to enable (`category`, `tag`, `recent_posts`)

### i18n

Translations in `i18n/en.yaml` and `i18n/ja.yaml`. The blog uses Japanese (`defaultContentLanguage: ja`).

## Verification Checklist for Visual Changes

Any change to SCSS or templates must be verified in a real browser before committing — a passing Hugo build only proves the SCSS compiles, not that the page looks right. Minimum bar:

1. Build clean: `cd ../.. && ./hugo --environment production` (run from the parent repo; `hugo` is not in PATH — the binary is `./hugo` on macOS, `./hugo.exe` on Windows).
2. Start `./hugo server --port 1313` and check with Playwright (or equivalent): emulate `colorScheme: 'light'` AND `'dark'` — both modes, always. Dark is defined as "amplified light"; a change that looks fine in one mode can break the other.
3. Pages to check: top page (cards, pagination), one article page (body must stay undecorated; TOC rail on articles with headings), one taxonomy page (widgets, pills), and mobile width (390px, single column).
4. Contrast: any new text color must reach WCAG AA 4.5:1 against `--background-color` in light mode (compute it, don't eyeball). Light-mode muted text is `#6E6167` for exactly this reason — the visually "nicer" lighter value failed AA.
5. Count always-on glows in one viewport: must stay ≤ 3 kinds (DESIGN.md §3.5).
6. Watch the browser console: no page errors, no 404 resources.

## Git Workflow

This theme is a submodule. Commit here first, then update the submodule reference in the parent repo (2-step commit). **Never `git push` without explicit user approval — pushing the parent repo deploys to production.**

## Formatting

- HTML templates: 2-space indent
- SCSS: 2-space indent (the actual convention in this codebase — older docs said 4-space; follow the surrounding code)
- Line endings: LF
