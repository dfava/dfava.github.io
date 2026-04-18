---
name: jekyll
description: Build, serve, and edit this Jekyll site. Use when adding posts, modifying pages, editing layouts/includes, changing navigation, or troubleshooting Jekyll builds.
---

# Jekyll Skill

## Serve and build

```bash
bundle exec jekyll serve          # serve at http://localhost:4000
bundle exec jekyll serve --drafts # include posts from _drafts/
bundle exec jekyll build          # build to _site/ without serving
```

`_config.yml` is not hot-reloaded — restart the server after editing it.

## Creating a new post

File goes in `_posts/` named `YYYY-MM-DD-title.markdown`. Required front matter:

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS +0000
---
```

Use `<!--more-->` to mark the excerpt break shown on the home page listing.

## Front matter reference

| Key | Effect |
|-----|--------|
| `layout` | `home`, `about`, `post`, `page`, `default` |
| `permalink` | Override the URL, e.g. `/about/` |
| `show_in_nav: false` | Hide page from the nav bar |

Posts dated in the future are suppressed because `future: false` is set in `_config.yml`.

## Theme overrides (minima)

- `_layouts/` — override a layout by creating a file matching the theme filename
- `_includes/` — override a partial by creating a file matching the theme filename
- `assets/css/custom.css` — custom styles, loaded globally via `_includes/head.html`
- Bootstrap 4 is loaded globally via `_includes/head.html`

## Navigation

The nav is rendered in `_includes/header.html` from `site.header_pages` (or all pages if unset). A page appears in the nav if it has a `title` in its front matter and `show_in_nav` is not `false`. To control nav order, set `header_pages` in `_config.yml` as an explicit list of page filenames.
