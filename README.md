# letingz.github.io

Personal academic site for Leting Zhang, built with Jekyll and the
[Minimal](https://github.com/pages-themes/minimal) GitHub Pages theme, restyled
onto a cream ground.

Built and served natively by GitHub Pages from `master` — no Actions workflow.

## Editing

- Pages are Markdown at the repo root: `index.md`, `research.md`, `cv.md`,
  `public-goods.md`.
- Sidebar, navigation, and portrait live in `_layouts/default.html`.
- All styling is `assets/css/style.scss`, which imports the theme and overrides
  the palette.

## Local preview

Requires Docker Desktop running.

```bash
docker run --rm -it -v "$PWD":/site -w /site -v letingz_gems:/usr/local/bundle \
  -p 4000:4000 ruby:3.3 bash -lc "bundle install --quiet && bundle exec jekyll serve --host 0.0.0.0"
```

Then open http://localhost:4000.

## Design docs

- Spec: `docs/superpowers/specs/2026-08-30-minimal-cream-rebuild-design.md`
- Plan: `docs/superpowers/plans/2026-08-30-minimal-cream-rebuild.md`
- The pre-rebuild site is preserved at the `academicpages-final` tag.
