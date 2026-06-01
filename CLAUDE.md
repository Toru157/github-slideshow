# CLAUDE.md

## Project Overview

`github-slideshow` is a Jekyll-based presentation site that renders markdown slide posts as a reveal.js slideshow. It was originally created as a GitHub Learning Lab exercise for teaching Git and GitHub workflows.

The site is designed to be hosted on GitHub Pages. Slides are Jekyll posts in `_posts/`, assembled into a single-page reveal.js presentation by `index.html`.

---

## Repository Structure

```
.
├── _config.yml          # Jekyll + reveal.js configuration
├── index.html           # Entry point; loops over posts to build slide deck
├── _posts/              # One Markdown file per slide (ordered by filename date)
│   └── 0000-01-01-intro.md
├── _layouts/
│   ├── presentation.html  # Outer shell for the full slideshow page
│   ├── slide.html         # Minimal single-slide layout
│   └── print.html         # Print/PDF layout
├── _includes/
│   ├── head.html          # <head> with reveal.js CSS links
│   ├── slide.html         # Renders a single <section> from a post
│   └── script.html        # reveal.js initialization script
├── node_modules/
│   └── reveal.js/         # Bundled via npm (not a submodule)
├── script/
│   ├── setup              # Install Ruby/gem dependencies
│   ├── server             # Run local Jekyll dev server
│   ├── cibuild            # Build + run htmlproofer (CI)
│   └── stage              # Deploy to internal GHE staging environment
├── Gemfile / Gemfile.lock # Ruby dependencies (github-pages, html-proofer)
├── package-lock.json      # npm lockfile (reveal.js)
├── .editorconfig          # Code style rules
└── .gitignore
```

---

## How Slides Work

- Every file in `_posts/` is a slide. Jekyll processes them in filename order (date prefix determines order).
- `index.html` iterates `site.posts` in reverse, including `_includes/slide.html` for each post.
- `_includes/slide.html` wraps each post in a `<section>` tag. The section gets:
  - `id` from the post's `slide-id` front matter (optional)
  - CSS classes from the `classes` front matter list; defaults to `step slide`
  - `data-*` attributes from the `data` front matter map (passed directly to reveal.js)

### Slide front matter conventions

```yaml
---
layout: slide          # always "slide"
title: "Slide Title"   # rendered as <h1>; set to "" to suppress
slide-id: my-anchor    # optional HTML id for deep-linking
classes:               # optional list of extra CSS classes
  - custom-class
data:                  # optional key/value pairs → data-* attributes
  background: "#fff"
---
```

---

## Development Setup

### Prerequisites
- Ruby (see `.ruby-version` if present; managed by rbenv on macOS)
- Bundler
- Node.js / npm (only needed to update `node_modules/reveal.js`)

### Install dependencies

```sh
script/setup
```

This installs gem dependencies via Bundler and initializes git submodules.

### Run the local dev server

```sh
script/server
# or equivalently:
bundle exec jekyll serve
```

The site is served at `http://localhost:4000` by default.

---

## Build & CI

```sh
script/cibuild
```

Runs `jekyll build --baseurl "."` then validates `_site/index.html` with `htmlproofer` (empty alt attributes are ignored). This is the canonical CI check — both steps must pass.

- Generated output lands in `_site/` (gitignored).
- `baseurl` in `_config.yml` is commented out intentionally for GitHub Pages compatibility; `script/cibuild` overrides it to `.` so local asset paths resolve during validation.

---

## Configuration (`_config.yml`)

Key settings:

| Key | Value | Notes |
|-----|-------|-------|
| `markdown` | `kramdown` | Use kramdown syntax in slide content |
| `highlighter` | `rouge` | Syntax highlighting |
| `timezone` | `Europe/Berlin` | Affects post date handling |
| `permalink` | `/:title` | URL slug derived from post title |
| `solarized.theme` | `dark` | Sets `<html class="dark">` for the Moon reveal.js theme |
| `plugins` | `jemoji` | GitHub emoji shortcodes work in post content |
| `reveal.*` | various | Passed to `Reveal.initialize()` — controls, transitions, sizing, etc. |

To change the reveal.js transition, edit `reveal.transition` (default: `linear`).  
To change slide dimensions, edit `reveal.width` / `reveal.height` (default: 1000×920).

---

## Adding a Slide

1. Create a file in `_posts/` named `YYYY-MM-DD-slug.md`.
2. Use a date that sorts where you want the slide to appear (earlier date = later in deck, because posts are iterated `reversed`).
3. Add the required front matter (at minimum `layout: slide` and `title`).
4. Write slide body in Markdown below the front matter.

---

## Code Style

Governed by `.editorconfig`:

| File type | Indent | Notes |
|-----------|--------|-------|
| `.md` / `.markdown` | 2 spaces | Trailing whitespace preserved; final newline required |
| `.json`, `.js`, `.css`, `.scss`, `.yml`, `.html` | 2 spaces | |
| All others | 4 tabs | |

Charset is UTF-8; line endings are LF.

---

## Deployment

### GitHub Pages (primary)
Push to `master`. GitHub Pages builds the Jekyll site automatically. No `baseurl` needed when the repo is served at the root of a GitHub Pages domain.

### Internal staging
```sh
script/stage [repo-name]
```
Builds with a staging `baseurl`, pushes `_site/` to `gh-pages` on an internal GHE instance (`training-staging/<repo>`), and opens the result.

---

## Key Constraints for AI Assistants

- **Do not commit `_site/`** — it is gitignored and rebuilt on each deploy.
- **Do not modify `node_modules/reveal.js/`** — it is managed by npm; update via `npm install reveal.js@<version>` and commit only `package-lock.json`.
- **Post filenames drive slide order** — if reordering slides, rename files; do not rely on front matter alone.
- **`baseurl` must remain commented out** in `_config.yml` for GitHub Pages to work; `script/cibuild` handles the local override.
- **The active development branch** is `claude/claude-md-docs-FciEz`. All changes should be developed and pushed there.
- **CI passes when** both `jekyll build` and `htmlproofer` exit cleanly. Always run `script/cibuild` to verify before pushing.
