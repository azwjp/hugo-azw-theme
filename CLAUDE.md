# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`hugo-azw-theme` is a custom Hugo theme for [blog.azw.jp](https://blog.azw.jp/). It is used as a git submodule from the parent blog repository at `../..` (the azw-blog root). The parent repo's CLAUDE.md has additional context about the overall blog setup.

**Design system**: All visual/styling work MUST follow `DESIGN.md` (Chromatic Neumorphism — design rules and the reasoning behind them). During the initial migration, `DESIGN-BOOTSTRAP.md` holds the one-time migration plan; it is deleted once migration is complete.

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

`baseof.html` is the root template. It uses Bulma's `.columns.is-desktop` for the two-column layout (main content + sidebar).

Key extension points for the parent site:
- `head.html` checks for `layouts/partials/myhead.html` in the parent repo — if it exists, it's injected at the end of `<head>`.
- `footer.html` checks for `layouts/partials/myfooter.html` in the parent repo — if it exists, it's injected after the footer.

### Styling

All custom CSS is in a single SCSS file: `assets/scss/style.scss`. It is compiled, minified, and fingerprinted by Hugo's asset pipeline (see `head.html` line 22).

**Dark mode** uses `@media (prefers-color-scheme: dark)` with CSS custom properties. Light mode SCSS variables (`$light-*`) are defined first, then dark mode values (`$dark-*`) are derived using SCSS `adjust-color()`. All colors are exposed as CSS custom properties (`--first-color`, `--background-color`, etc.) and re-wrapped into SCSS variables (`$firstColor`, `$backgroundColor`, etc.) for use throughout the file.

**Neumorphism design**: The theme uses neumorphic box-shadow patterns extensively. Key shadow variables: `$boxShadowColor`, `$boxShadowHighlightColor`, `$boxShadowEnlightColor`. When adding new UI elements, follow the existing shadow pattern (positive offset + negative offset + optional inset).

**Responsive breakpoints** (SCSS mixins): `@include smaller` (<768px), `@include middle` (768–1216px), `@include larger` (≥768px), `@include wide` (≥1216px).

### External Dependencies (CDN)

Loaded in `head.html` — not bundled:
- Bulma 0.9.0 (CSS framework)
- FontAwesome 6.4.2
- jQuery 3.4.1
- Lightbox2 2.7.1 (image galleries)
- Tocbot 4.11.1 (table of contents)

### Configuration

See `exampleSite/config.yml` for all supported params. Key theme-specific params:
- `params.site.header` — header image path
- `params.site.deafult_thumbnail` — fallback thumbnail (note: the typo "deafult" is intentional/legacy)
- `params.widgets` — list of sidebar widgets to enable (`category`, `tag`, `recent_posts`)

### i18n

Translations in `i18n/en.yaml` and `i18n/ja.yaml`. The blog uses Japanese (`defaultContentLanguage: ja`).

## Formatting

- HTML templates: 2-space indent
- SCSS/other files: 4-space indent
- Line endings: LF
