# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Serve locally (with drafts):**
```
bin/serve
# or: bundle exec jekyll serve --drafts
```

**Process webring images:**
```
npm run images
```

## Architecture

This is a Jekyll static site for [codingitwrong.com](https://codingitwrong.com), built with a custom GitHub-hosted theme (`jekyll-theme-codingitwrong`).

### Content types

- **`_posts/`** — Blog posts in Markdown, named `YYYY-MM-DD-slug.md`. Front matter supports `title`, `tags`, and `comments: false` to disable the comment section.
- **`_drafts/`** — Unpublished posts, rendered when serving with `--drafts`.
- **`streams/`** — Longer tutorial/series pages.
- **`tag/`** — One HTML file per tag (e.g., `tag/testing.html`) using the `tag` layout to list posts with that tag.
- Static pages at the root (e.g., `books.md`, `projects.md`, `speaking.md`).

### Layouts and includes

- Layouts in `_layouts/` (`page`, `post`, `tag`) extend the theme's `default` layout.
- `_includes/` has reusable partials: `comments.html`, `follow-link.html`, `video-thumbnail.html`, book display components, and tutorial content fragments.

### Theme

The site uses a custom gem-based theme sourced directly from GitHub (`codingitwrong/jekyll-theme-codingitwrong`, `main` branch). Changes to global styles or base layouts require editing that separate repository.

### Deployment

The site deploys to GitHub Pages via GitHub Actions. Pushing to `main` triggers the workflow at `.github/workflows/deploy.yml`, which builds the site with `JEKYLL_ENV=production` and deploys it.
