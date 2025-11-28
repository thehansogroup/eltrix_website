# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based documentation website for Eltrix, an Elixir-powered Matrix homeserver. Deployed automatically to GitHub Pages on push to `main`.

## Development Commands

```bash
# Install dependencies
bundle install

# Start development server
bundle exec jekyll serve

# Start with live reload
bundle exec jekyll serve --livereload

# Build for production
JEKYLL_ENV=production bundle exec jekyll build
```

The dev server runs at `http://localhost:4000`. Built files go to `_site/`.

## Architecture

- **Layouts**: `_layouts/default.html` (main), `_layouts/doc.html` (documentation pages)
- **Includes**: `_includes/nav.html`, `footer.html`, `doc-sidebar.html`
- **Documentation**: Markdown files in `_docs/` using the `doc` layout
- **Styling**: Tailwind CSS via CDN with custom config in `default.html`

## Documentation Front Matter

All docs in `_docs/` require:

```yaml
---
title: Page Title
description: Brief description
category: getting-started  # getting-started | deployment | advanced | reference
order: 1                   # Sort order within category
nav_title: Short Title     # Optional, for sidebar
---
```

## Design System

- **Colors**: Dark theme with `#0d0d15` background, `#a855f7` purple accent, `#06b6d4` cyan for links
- **Fonts**: IBM Plex Sans (body), IBM Plex Mono (code)
- **Visual effects**: Scanlines, noise texture, grid patterns defined in `default.html`
