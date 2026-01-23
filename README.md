# someguys.blog

[![Netlify Status](https://api.netlify.com/api/v1/badges/746dc785-842c-4593-bcbc-db7395b89a52/deploy-status)](https://app.netlify.com/sites/someguysblog/deploys)

A personal tech blog built with [Hugo](https://gohugo.io/) and the [beautifulhugo](https://github.com/halogenica/beautifulhugo) theme.

## Setup

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/abtreece/someguys.blog.git

# Or if already cloned
git submodule update --init --recursive
```

## Local Development

```bash
# Run dev server with drafts
hugo server -D

# Build for production
hugo --gc --minify
```

## Creating Posts

```bash
hugo new posts/YYYY-MM-DD-post-title.md
```

Posts use YAML front matter with `layout`, `title`, `date`, and optional `tags`.

## Deployment

Automatically deployed to [someguys.blog](https://someguys.blog) via Netlify on push to main.
