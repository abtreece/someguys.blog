# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Hugo static site blog ("Some Guys Blog") deployed on Netlify. The site uses the beautifulhugo theme as a git submodule.

## Commands

```bash
# Local development server (with drafts)
hugo server -D

# Build the site
hugo --gc --minify

# Create new post
hugo new posts/YYYY-MM-DD-post-title.md
```

## Project Structure

- `content/posts/` - Blog posts (use date prefix: `YYYY-MM-DD-title.md`)
- `content/pages/` - Static pages (e.g., talks.md)
- `static/img/` - Images and static assets
- `layouts/partials/` - Theme overrides
- `hugo.toml` - Site configuration
- `netlify.toml` - Netlify build and deploy settings

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
- Theme: beautifulhugo
- Author info and social links in `[params.author]`
- Menu items in `[[menu.main]]`

## Deployment

- Hosted on Netlify with automatic deploys from main branch
- Hugo version: 0.154.5 (defined in netlify.toml)
- Build command: `hugo --gc --minify`
- Output directory: `public/`

## Theme

The beautifulhugo theme is included as a git submodule at `themes/beautifulhugo`. After cloning, run:
```bash
git submodule update --init --recursive
```

Theme customizations go in `layouts/` to override theme templates.

## Git Commits

- Use conventional commits: `feat:`, `fix:`, `docs:`, `chore:`
- Keep commit messages concise and descriptive
- Do NOT add Co-Authored-By lines to commits
