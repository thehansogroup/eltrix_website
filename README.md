# Eltrix Website

The official website and documentation for [Eltrix](https://github.com/eltrix/eltrix), the Elixir-powered Matrix homeserver.

## Local Development

### Prerequisites

- Ruby 3.3+
- Bundler

### Setup

```bash
# Install dependencies
bundle install

# Start the development server
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`.

### Live Reload

For automatic browser refresh during development:

```bash
bundle exec jekyll serve --livereload
```

## Project Structure

```
.
├── _config.yml          # Jekyll configuration
├── _layouts/            # Page layouts
│   ├── default.html     # Main layout
│   └── doc.html         # Documentation layout
├── _includes/           # Reusable components
│   ├── nav.html         # Navigation header
│   ├── footer.html      # Footer
│   └── doc-sidebar.html # Docs sidebar
├── _docs/               # Documentation pages (markdown)
├── assets/              # Static assets
│   └── images/          # Images and logo
├── index.html           # Landing page
├── privacy.md           # Privacy policy
└── imprint.md           # Imprint/legal
```

## Writing Documentation

Documentation pages live in `_docs/` and use the `doc` layout. Each page needs front matter:

```yaml
---
title: Page Title
description: Brief description
nav_title: Short Nav Title  # Optional, for sidebar
category: getting-started   # getting-started, deployment, advanced, or reference
order: 1                    # Sort order within category
---
```

### Categories

- `getting-started` - Introduction and basic setup
- `deployment` - Docker, Kubernetes deployment guides
- `advanced` - Scaling, federation, advanced topics
- `reference` - API documentation

## Deployment

The site automatically deploys to GitHub Pages when changes are pushed to `main`.

### Manual Deployment

1. Ensure GitHub Pages is configured to use GitHub Actions
2. Push to `main` branch
3. The workflow at `.github/workflows/jekyll.yml` handles the rest

### Build for Production

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

The built site will be in `_site/`.

## Design System

### Colors

| Name | Hex | Usage |
|------|-----|-------|
| Background | `#09090b` | Main background |
| Secondary | `#0c0c10` | Cards, code blocks |
| Tertiary | `#18181b` | Hover states |
| Border | `#27272a` | Borders, dividers |
| Text | `#e4e4e7` | Primary text |
| Muted | `#71717a` | Secondary text |
| Purple | `#a855f7` | Primary accent |
| Cyan | `#06b6d4` | Links, highlights |
| Blue | `#38bdf8` | Secondary accent |

### Typography

- **Headings**: IBM Plex Sans (semibold/bold)
- **Body**: IBM Plex Sans (regular)
- **Code**: IBM Plex Mono

### Components

The site uses Tailwind CSS via CDN with custom configuration. Key components:

- Bordered cards (not glowy)
- Terminal-style code blocks
- Monospace labels and badges
- Subtle scanline/noise textures

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with `bundle exec jekyll serve`
5. Submit a pull request

## License

This website is part of the Eltrix project and is licensed under the Apache License 2.0.
