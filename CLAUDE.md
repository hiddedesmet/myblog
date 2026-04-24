# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

All commands should be run from the `myblog/` directory.

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve --livereload

# Build for production
bundle exec jekyll build

# Serve with drafts visible
bundle exec jekyll serve --drafts
```

The generated site outputs to `_site/` — never edit files there directly.

## Architecture

This is a Jekyll static site blog at [hiddedesmet.com](https://hiddedesmet.com), using the Nomod theme.

### Configuration

- **`_config.yml`** — Jekyll core config: URL, permalink style (`/:title`), plugins, collection definitions, and layout defaults.
- **`_data/settings.yml`** — All site-wide content and feature flags: navigation menu, hero section, social links, footer text, Mailchimp, Disqus, Google Analytics, and contact form settings. This is the primary file to edit for site-wide changes.

### Content Collections

| Directory | Purpose | Layout |
|-----------|---------|--------|
| `_posts/` | Blog posts (filename: `YYYY-MM-DD-slug.md`) | `post` |
| `_pages/` | Static pages (about, tags, featured, videos) | `page` |
| `_authors/` | Author profiles | `author` |

### Post Front Matter

```yaml
---
layout: post
title: "Post Title"
description: "SEO description"
date: 2025-01-01 10:00:00 +0300
author: hidde          # matches filename in _authors/
image: '/images/cover.jpg'
tags: [AI, Development]
featured: false        # shows in featured section
toc: true              # enables table of contents
---
```

### Layouts & Includes

- **`_layouts/`** — Base templates: `default`, `page`, `post`, `author`, `contact`
- **`_includes/`** — Reusable partials for header, footer, sections (hero, posts, featured, authors, subscribe), share buttons, TOC, Disqus comments, and Google Analytics

### Styling

Sass lives in `_sass/`. The entry point is `_includes/main.scss`. The site supports `auto`, `light`, and `dark` color schemes (set in `_data/settings.yml` → `color_scheme`).

### Images

Store post images in `images/`. Reference them as `/images/filename.jpg` in front matter and content.

### Pagination

Configured to 6 posts per page via `jekyll-paginate`. Path pattern: `/page/:num`.
