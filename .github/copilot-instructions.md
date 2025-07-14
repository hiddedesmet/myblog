# Copilot Instructions for Hidde's Technical Blog

## Architecture Overview
This is a Jekyll-based technical blog hosted at hiddedesmet.com, focusing on Azure, AI, DevOps, and Infrastructure as Code content. The site uses a custom theme with modular SASS architecture and follows Jekyll's convention-over-configuration approach.

## Key Project Structure
- **Content**: `_posts/` (blog articles), `_authors/` (author profiles), `_pages/` (static pages)
- **Templates**: `_layouts/` (page templates), `_includes/` (reusable components)  
- **Configuration**: `_config.yml` (Jekyll config), `_data/settings.yml` (theme settings)
- **Styling**: `_sass/` organized as `0-settings/`, `1-tools/`, `2-base/`, `3-modules/`, `4-layouts/`
- **Generated**: `_site/` (build output, don't edit directly)

## Content Creation Patterns

### Blog Posts
Posts follow naming convention: `YYYY-MM-DD-post-slug.md` in `_posts/`

Required frontmatter:
```yaml
layout: post
title: "Post Title"
description: "SEO-friendly description"
date: YYYY-MM-DD HH:MM:SS +0000
author: hidde
image: '/images/featured-image.jpg'
tags: [tag1, tag2]
featured: false  # true for homepage featured section
toc: true       # enables table of contents
```

### Author Management
Authors defined in `_authors/username.md` with profile info. Posts reference author by `username` field.

### Navigation & Menus
Site navigation configured in `_data/settings.yml` under `menu_settings`. Supports nested submenus for resource organization.

## Development Workflow

### Local Development
```bash
bundle install          # Install dependencies
bundle exec jekyll serve # Start dev server (localhost:4000)
bundle exec jekyll build # Generate static site
```

### Content Guidelines
- **Technical Focus**: Posts cover Azure, AI/ML, DevOps, IaC (Bicep/Terraform)
- **Code Examples**: Use fenced code blocks with language specification
- **Images**: Store in `/images/`, reference as `/images/filename.jpg`
- **Table of Contents**: Enable with `toc: true` for long-form technical content

## Theme Customization

### Settings Configuration
Core site settings in `_data/settings.yml`:
- Site metadata (title, description, logo)
- Navigation structure and CTAs
- Social links and contact forms
- Feature toggles (hero, newsletter, etc.)

### Styling Architecture
SASS organized by purpose:
- `0-settings/`: Variables, colors, typography
- `1-tools/`: Mixins and functions  
- `2-base/`: Reset, typography, base elements
- `3-modules/`: Component styles
- `4-layouts/`: Page-specific layouts

### Include Components
Reusable components in `_includes/`:
- `article.html`: Post content wrapper
- `section-*.html`: Homepage sections
- `share-buttons.html`: Social sharing
- `toc.html`: Table of contents generation

## Deployment & Performance
- Static site generation optimized for performance
- Lazy loading enabled for images (`class="lazy"`)
- Responsive design with mobile-first approach
- SEO optimized with meta tags and structured data

## Common Tasks
- **New post**: Create in `_posts/` with proper frontmatter
- **New page**: Add to `_pages/` and update navigation in settings.yml
- **Style changes**: Modify appropriate SASS files in `_sass/`
- **Layout updates**: Edit templates in `_layouts/` or components in `_includes/`
