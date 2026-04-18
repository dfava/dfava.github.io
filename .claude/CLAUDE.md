# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bundle install          # install dependencies
bundle exec jekyll serve  # serve locally at http://localhost:4000
```

## Architecture

This is a personal academic/blog website built with Jekyll, using the `minima` theme with customizations. It deploys to GitHub Pages at [danielfava.com](http://danielfava.com).

**Content files** at the root (`about.markdown`, `cv.markdown`, `publications.markdown`, `talks.markdown`, `teaching.markdown`, `service.markdown`, `news.markdown`, `index.markdown`) are static pages using front matter to set layout and permalink.

**Blog posts** live in `_posts/` as dated Markdown files (`YYYY-MM-DD-title.markdown`). The `excerpt_separator: <!--more-->` in `_config.yml` controls what appears on the home page listing.

**Theme overrides** — the minima theme is extended via:
- `_layouts/home.html` and `_layouts/about.html` — override minima's defaults
- `_includes/head.html` — injects Bootstrap 4 and a custom CSS file (`assets/css/custom.css`)

Posts with `future: false` in `_config.yml` will not appear until their date is reached.