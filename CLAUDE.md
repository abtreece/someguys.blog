# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Hugo static site blog ("Some Guys Blog") deployed on Netlify. The site uses the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme as a git submodule, with project-level overrides in `layouts/_partials/` and `assets/css/extended/`.

## Commands

```bash
# Local development server (with drafts)
hugo server -D

# Build the site (Netlify build command)
hugo --gc --minify

# Create new post
hugo new posts/YYYY-MM-DD-post-title.md
```

## Project Structure

- `content/posts/` - Blog posts (use date prefix: `YYYY-MM-DD-title.md`)
- `static/img/` - Images and other static assets
- `static/fonts/` - Self-hosted Inter and JetBrains Mono woff2 files
- `layouts/_partials/` - PaperMod partial overrides (e.g., `footer.html`, `home_info.html`)
- `assets/css/extended/` - Custom CSS layered on top of PaperMod's bundled styles; files are concatenated alphabetically, so prefix filenames numerically when order matters (`01-fonts.css`, `02-tokens.css`, etc.)
- `hugo.toml` - Site configuration
- `netlify.toml` - Netlify build and deploy settings
- `docs/` - Untracked design and handoff notes (`design-system.md`, `claude-code-handoff.md`, `hero-copy-handoff.md`)

## Post Front Matter

Posts use YAML front matter:
```yaml
---
layout: post
title: Post Title
date: YYYY-MM-DD
tags: ["tag1", "tag2"]
---
```

Use `<!--more-->` to define the excerpt break point.

## Configuration

Key settings in `hugo.toml`:
- Theme: PaperMod (via submodule at `themes/PaperMod`)
- `defaultTheme = "auto"` for system-preference dark/light with manual toggle persisted in localStorage
- `params.label.text` controls the wordmark; the trailing accent period is appended via CSS (`.logo a::after`)
- `params.homeInfoParams.Content` is the home-page hero copy
- `params.socialIcons` lists footer icons (GitHub, LinkedIn, X, Bluesky, RSS)
- `[[menu.main]]` entries control header nav

## Design system

The visual identity is documented in `docs/design-system.md` (untracked). Key constraints:

- Inter (400/500/600/700) for text, JetBrains Mono (400) for code, all self-hosted from `static/fonts/`
- Color tokens for both themes are defined in `assets/css/extended/02-tokens.css`, mapped onto PaperMod's `--theme/--entry/--primary/--secondary/--tertiary/--content/--code-bg/--border` variables
- Accent (ochre) appears only on links, post tags, code syntax, and the wordmark trailing period; nowhere else
- No em-dashes (`—` or `--`) in body copy

## Deployment

- Hosted on Netlify with automatic deploys from `main`
- Hugo version: 0.161.1 (defined in `netlify.toml`)
- Build command: `hugo --gc --minify`
- Output directory: `public/`

## Theme

PaperMod is a git submodule at `themes/PaperMod`. After cloning, run:

```bash
git submodule update --init --recursive
```

Customizations live in the project, not the submodule:

- **Templates** override at `layouts/_partials/<name>.html` (PaperMod uses Hugo's `_partials` layout structure introduced in 0.146)
- **CSS** overrides go in `assets/css/extended/*.css`; they are concatenated and minified after PaperMod's bundled styles
- Never edit files inside `themes/PaperMod/` directly

Note that overriding `_partials/footer.html` requires preserving PaperMod's bundled scripts (theme toggle, scroll-to-top, smooth-anchor scroll, code-copy buttons) since they live alongside the footer markup in that file.

## Git Commits

- Use conventional commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`
- Keep commit messages concise and descriptive
- Do NOT add Co-Authored-By lines to commits
- Use worktrees at `.worktrees/<branch>` for any task with clear boundaries; one PR per independent task
