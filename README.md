# myblog

Source for [sdeaton.com](https://sdeaton.com) — a personal blog on AI, developer tools, technical writing, and whatever else catches my interest.

## Tooling

- [Zola](https://www.getzola.org/) — a static site generator written in Rust. Posts are Markdown in `content/blog/`, pages live in `content/pages/`, and site config is in `config.toml`.
- [Linkita](https://codeberg.org/salif/linkita) — the theme, pulled in as a git submodule at `themes/linkita`.
- Sass for styling (compiled by Zola at build time) and a built-in search index.

## Layout

- `content/` — Markdown posts and pages.
- `templates/` — site-level template overrides on top of the theme.
- `static/` — files served as-is (favicon, custom CSS).
- `themes/linkita/` — theme submodule.
- `config.toml` — site config (base URL, menu, taxonomies, markdown options).
- `public/` — Zola build output, gitignored.

## Setup

Clone with submodules so the theme is included:

```sh
git clone --recurse-submodules <repo-url>
cd myblog
```

If you already cloned without `--recurse-submodules`:

```sh
git submodule update --init --recursive
```

Install Zola (macOS):

```sh
brew install zola
```

For other platforms, see the [Zola installation docs](https://www.getzola.org/documentation/getting-started/installation/).

## Run locally

Start the dev server with live reload:

```sh
zola serve
```

The site is served at `http://127.0.0.1:1111` and rebuilds on file changes.

To preview drafts (posts with `draft = true` in their frontmatter):

```sh
zola serve --drafts
```

## Build

Generate the production site into `public/`:

```sh
zola build
```

Check for broken internal links and other issues:

```sh
zola check
```

## Adding a post

Create a new directory under `content/blog/` with an `index.md`:

```
content/blog/my-new-post/index.md
```

Frontmatter template:

```toml
+++
title = "Post title"
date = 2026-04-26
description = "Short description used in feeds and previews."

[taxonomies]
tags = ["tag-one"]
categories = ["category-one"]
+++
```

Co-locate any images for the post in the same directory and reference them with relative paths.

## Updating the theme

```sh
git submodule update --remote themes/linkita
```
