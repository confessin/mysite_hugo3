# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal portfolio + blog of Mohammad Rafi, deployed at https://mrafi.in via Netlify. It is a
customized fork of the [hugo-profile](https://github.com/gurusabarish/hugo-profile) Hugo theme.
The repo is dual-purpose: the **root is the theme**, and **`exampleSite/` is the actual live site**
that consumes it.

## Repo layout (the key thing to understand)

Two layers live in one repo:

- **Root = the theme** — `layouts/`, `static/` (Bootstrap 5, FontAwesome 5, theme CSS, decorative
  images), `archetypes/`, `theme.toml`.
- **`exampleSite/` = the deployed site** — `config.yaml` (all site content + settings),
  `content/` (blog posts, standalone pages), `static/` (Rafi's own images and JS).

Netlify builds `exampleSite/` using the repo root as its theme (`--themesDir ../..`). The
root-level `/config.yaml`, `/content`, and `/static/images` are **gitignored** — they are not where
the site lives. The real, committed config is always **`exampleSite/config.yaml`**.

## Editing site content

The homepage is a single page assembled by `layouts/index.html` from section partials in
`layouts/partials/sections/` (hero → about → experience → education → projects → achievements →
contact). **Each section is driven entirely by `params` in `exampleSite/config.yaml`, not by content
files.** Updating the resume, skills, projects, etc. means editing `exampleSite/config.yaml` — this
is what nearly every recent commit does.

- **Experience / education / achievements / projects** are lists under `params.experience.items`,
  `params.education.items`, etc. Each section (and most sub-features) has an `enable:` toggle.
- **Projects** come from two sources, both rendered by `layouts/partials/sections/projects.html`:
  inline `params.projects.items` (current approach), and page-based projects in `content/projects/`
  (pages with `Type: projects`) if any exist.
- **Blog posts** live in `exampleSite/content/blogs/*.md` with standard Hugo front matter.
  `markup.goldmark.renderer.unsafe: true` is set, so raw HTML inside markdown is allowed.
- **Images** referenced in config by root-absolute path (e.g. `/base.png`, `/work_rafi.png`) live in
  `exampleSite/static/`; theme/decorative images live in the root `static/`.
- A standalone scrollorama "wedding" page is `exampleSite/content/wedding.html`, with its assets
  under `exampleSite/static/js/ecard/`.

## Build & run

Hugo is the only toolchain — there are no tests, linters, or package manifests. It is pinned to
**0.160.1** (`netlify.toml`; set the same value as `HUGO_VERSION` in the Cloudflare Pages project).
Run all commands from `exampleSite/`.

```bash
# Production build (what Netlify runs) — outputs to exampleSite/public/
cd exampleSite && hugo --gc --minify --themesDir ../..

# Local dev server
cd exampleSite && hugo server --themesDir ../.. --theme mysite_hugo3
```

`--theme mysite_hugo3` is required locally because `config.yaml` sets `theme: hugo-profile`, which
does not match this repo's folder name; Netlify sidesteps this by setting `HUGO_THEME=repo`.

### Hugo compatibility notes

The site builds clean on current Hugo (verified on 0.160.1). Three theme pieces were adapted for
Hugo ≥0.120/0.156:

- `layouts/partials/head.html` uses `_internal/google_analytics.html` (the `_async` variant was
  removed in Hugo 0.120).
- `layouts/shortcodes/gist.html` and `layouts/shortcodes/tweet.html` are local compatibility shims
  for the `gist` and `tweet` shortcodes Hugo removed in 0.156 (used by the demo post
  `exampleSite/content/blogs/rich-content.md`). They render client-side embeds, so there are no
  build-time calls to GitHub/X APIs.

Analytics is wired through the legacy `googleAnalytics: "UA-..."` key — a dead Universal Analytics
property — so the GA template emits nothing on modern Hugo. Set a GA4 `G-...` ID to re-enable.
