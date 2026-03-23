# Hugo Azw Theme

A custom Hugo theme with a neumorphic design, dark mode support, and a responsive two-column layout built on [Bulma](https://bulma.io/).

**Demo**: [blog.azw.jp](https://blog.azw.jp/)

## Features

- Neumorphic UI with soft box-shadow styling
- Automatic dark mode via `prefers-color-scheme`
- Responsive two-column layout (main content + sidebar)
- Table of contents (via Tocbot) on single pages, displayed in a left sidebar on wide screens
- Image lightbox (via Lightbox2)
- Social sharing buttons (Twitter, Facebook, Hatena Bookmark, Pocket, LINE)
- Hierarchical category widget with nested display
- Tag cloud and recent posts sidebar widgets
- Featured image support with automatic resizing
- i18n support (English and Japanese included)
- Google Analytics integration
- Custom partial injection points for parent sites (`myhead.html`, `myfooter.html`)

## Requirements

- Hugo **extended** edition (required for SCSS compilation)

## Installation

Add the theme as a git submodule:

```bash
cd your-hugo-site
git submodule add https://github.com/azwjp/hugo-azw-theme.git themes/hugo-azw-theme
```

Or clone it directly into your `themes/` directory.

## Configuration

Below is a full annotated `config.yml`. Copy the parts you need into your site's `config.yml`.

```yaml
baseURL: https://example.com/
theme: hugo-azw-theme
title: My Blog

# Language & content settings
defaultContentLanguage: ja          # or "en"
hasCJKLanguage: true                # Enable CJK language support
paginate: 7
enableEmoji: true

# Optional: Google Analytics
googleAnalytics: "UA-XXXXXXXX-X"

# Permalink structure
permalinks:
  posts: /:year/:month/:contentbasename/

# Markdown rendering
markup:
  goldmark:
    renderer:
      unsafe: true                  # Allow raw HTML in Markdown

# Taxonomies (all three are supported by the theme)
taxonomies:
  category: categories
  tag: tags
  series: series
```

### Menus

The theme supports two menus: `main` (rendered in the header) and `footer`.

```yaml
menu:
  main:
    # Simple link
    - identifier: home
      name: Blog Top
      title: Blog Top
      url: /
      weight: 1

    # Link with FontAwesome icon, opens in a new tab
    - identifier: web
      name: About
      title: About
      url: https://example.com/
      weight: 2
      params:
        icon: fa-solid fa-globe     # Any FontAwesome 6 class
        target: _blank              # Opens in a new tab

  footer:
    - identifier: disclaimer
      name: Disclaimer
      title: Disclaimer
      url: /disclaimer/
      weight: 1
```

**Menu item params:**

| Param    | Description                                        |
|----------|----------------------------------------------------|
| `icon`   | FontAwesome 6 icon class (e.g. `fa-solid fa-globe`). Displayed before the link text. |
| `target` | Link target attribute (e.g. `_blank`). When set to `_blank`, `rel="noopener noreferrer"` is added automatically. |

### Theme Parameters (`params`)

```yaml
params:
  # -- General --
  since: 2017                       # Copyright start year (displayed in footer)
  dateFormat: "2006-01-02"          # Go date format for post dates
  enablePostCopyright: true         # Show copyright notice on posts
  mainSections: ["posts"]           # Content sections to list on the homepage
                                    # Defaults to ["post"] if omitted

  # -- Site appearance --
  site:
    title: Hugo                     # Site title (used in the header area)
    header: "/header-img.jpg"       # Header image on the homepage (index page)
    # headerSingle: "/header-single.jpg"  # Optional: separate header image for single pages
    deafult_thumbnail: "no-thumbnail.jpg" # Fallback thumbnail for posts without images
                                          # (Note: the key "deafult" is a legacy typo)
    image_size_thumbnail: "640x q60"      # Hugo image processing spec for thumbnails
    image_size_preview: "1200x q80"       # Hugo image processing spec for preview images
    listSize: 5                           # Number of posts per page on the homepage

  # -- Sidebar widgets --
  widgets:
    - category                      # Hierarchical category list
    - tag                           # Tag cloud
    - recent_posts                  # Recent posts with thumbnails
```

### Parameters Reference

#### `params.site`

| Key                    | Type   | Description |
|------------------------|--------|-------------|
| `title`                | string | Displayed as the site title. |
| `header`               | string | Path to the header image shown on the homepage. |
| `headerSingle`         | string | Path to the header image shown on single (post) pages. If omitted, no header image is shown on single pages. |
| `deafult_thumbnail`    | string | Fallback thumbnail image filename when a post has no `featured_image`. |
| `image_size_thumbnail` | string | Hugo [image processing](https://gohugo.io/content-management/image-processing/) spec for thumbnail generation (e.g. `"640x q60"`). |
| `image_size_preview`   | string | Hugo image processing spec for preview/featured images (e.g. `"1200x q80"`). |
| `listSize`             | int    | Number of posts displayed per page on the homepage. |

#### `params.widgets`

An ordered list of sidebar widgets to display. Available widgets:

| Widget         | Description |
|----------------|-------------|
| `category`     | Hierarchical category tree with post counts. Categories are nested based on the order they appear in each post's `categories` front matter. |
| `tag`          | Flat list of all tags with post counts. |
| `recent_posts` | The 5 most recent posts with thumbnails and dates. |

### Front Matter

Posts support the following front matter fields:

```yaml
---
title: "My Post Title"
date: 2024-01-15
categories:
  - Tech                      # Categories are hierarchical (order matters)
  - Hugo
tags:
  - hugo
  - blogging
featured_image: "cover.jpg"   # Page bundle resource or absolute path
images:
  - "/path/to/og-image.jpg"   # Used for OpenGraph/Twitter cards; first image
                               # is also used as thumbnail if no featured_image
description: "A short description for meta tags and summaries."
---
```

**Image resolution for `featured_image`:**
- If the post is a [page bundle](https://gohugo.io/content-management/page-bundles/) and the image is a page resource, Hugo will automatically resize it using `image_size_preview` / `image_size_thumbnail`.
- If the image is an absolute path (e.g. `/images/cover.jpg`), it is used as-is without resizing.
- If neither `featured_image` nor `images` is set, a placeholder image (`noimage.png`) is shown in list views.

## Shortcodes

### `image`

Display an image with a caption and lightbox support.

```
{{</* image src="/path/to/image.jpg" caption="An example image" */>}}
```

| Param     | Description |
|-----------|-------------|
| `src`     | Image URL or path. |
| `caption` | Caption text (HTML allowed). |

### `table`

Wrap a Markdown table with a caption.

```
{{</* table caption="Table 1: Example data" */>}}
| Column A | Column B |
|----------|----------|
| foo      | bar      |
{{</* /table */>}}
```

| Param     | Description |
|-----------|-------------|
| `caption` | Caption text displayed above the table (HTML allowed). |

## Customization

### Injecting Custom HTML

The theme provides two extension points for the parent site:

1. **`layouts/partials/myhead.html`** - If this file exists in your site, its contents are injected at the end of `<head>`. Use it for custom fonts, additional CSS, or meta tags.

2. **`layouts/partials/myfooter.html`** - If this file exists in your site, its contents are injected after the `<footer>`. Use it for additional scripts (e.g. syntax highlighting).

### Styling

All theme styles live in `assets/scss/style.scss`. The theme uses:

- **Bulma 0.9.0** as the CSS framework (loaded via CDN)
- **SCSS custom properties** for color theming and dark mode
- **Neumorphic box-shadows** for the distinctive soft UI appearance

### External Dependencies (CDN)

The following libraries are loaded from CDNs:

| Library       | Version | Purpose              |
|---------------|---------|----------------------|
| Bulma         | 0.9.0   | CSS framework        |
| FontAwesome   | 6.4.2   | Icons                |
| jQuery        | 3.4.1   | DOM manipulation     |
| Lightbox2     | 2.7.1   | Image lightbox       |
| Tocbot        | 4.11.1  | Table of contents    |

## i18n

Translation files are in the `i18n/` directory. English (`en.yaml`) and Japanese (`ja.yaml`) are included. Set `defaultContentLanguage` in your config to choose the language.

## License

[MIT](LICENSE)
