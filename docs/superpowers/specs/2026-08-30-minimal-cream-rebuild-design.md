# Rebuilding letingz.github.io on Minimal + cream

**Date:** 2026-08-30
**Status:** awaiting review
**Repo:** `letingz.github.io` (branch `master`, served by GitHub Pages)

## Why

The site's visible problem is that most of its content is not the owner's. Of 385
tracked files, roughly 20 are Leting Zhang's. The rest arrived with a fork:

- `_publications/` — four cognitive-psychology / ERP papers by Y. Cheng, with
  download links pointing at `lilianyou.github.io`
- `_posts/` — 24 files, every one named `blog-post-1.md`
- `_talks/` — four placeholders (`UC San Francisco, Department of Testing`,
  `London School of Testing`)
- ~23 Academic Pages demo pages, plus `markdown_generator/` and `talkmap/`

`_pages/resources.md` — linked from the nav as "Public Goods" — reads in full:
`# Resource 1` / `# Resource 2`. The CV PDF dates to August 2022. The last
content commit was 2024-08-25.

A theme migration does not fix any of that; the inherited files follow into any
theme. But the content has to be rewritten either way, so doing it during a move
costs little extra and produces a site the owner can maintain.

## Decisions

Each of these was chosen against alternatives that were mocked up and rejected.

| Decision | Chosen | Rejected, and why |
|---|---|---|
| Base theme | `pages-themes/minimal` | **al-folio** — 350 files, needs `jekyll/scholar`, which GitHub Pages will not build, forcing an Actions pipeline and a Ruby 3 upgrade. **Academic Pages** (status quo) — the fork that caused the problem. |
| Publication list | Hand-written HTML/Markdown | **BibTeX + jekyll-scholar** — explicitly ruled out by the owner. Needs the Actions build. |
| Ground colour | `#faf8f5` (from paulgp.com) | Minimal's stock `#ffffff` — compared side by side via a toggle in the mockup. |
| Sidebar | Boxed, `#f5f2ed` / 8px radius / 1.5rem padding (from paulgp.com) | Minimal's unboxed sidebar; and Minimal's `.downloads` button strip, which nested a box inside a box. |
| Typography | Minimal's own — Noto Sans 14px/1.5 | paulgp.com's Spectral + DM Sans — would have made the site a copy rather than a relative. |
| Pages | Home, Research, Papers, CV, Public Goods | Folding the research narratives into the Papers page — rejected 2026-08-30; the deep dives get their own page. |

**Approved mockup:** https://claude.ai/code/artifact/be74b4b6-1bd9-4fc7-87a4-22e6345d14ac

### Why this base is cheap

`jekyll-theme-minimal` is on GitHub Pages' supported-theme list, so the existing
native build keeps working. No GitHub Actions workflow, no `gh-pages` branch, no
Pages source change, and no Ruby 3 upgrade (local Ruby is currently 2.6.10 with a
`.ruby-version` of 2.7.0 and a non-resolving `bundle`; this only affects optional
local preview, not deployment).

The entire custom layer is one `_config.yml` line, one overridden layout, and
roughly 40 lines of CSS.

## Non-goals

- No blog. `_posts/` is deleted and not replaced.
- No dark mode. Committed single light palette, as paulgp.com does.
- No talks, teaching, or portfolio pages.
- No JavaScript beyond Minimal's bundled `scale.fix.js`.
- No CI, linting, or link-checking workflows.
- Writing the actual publication list is **out of scope for implementation** —
  see Open question 2.

## Target structure

Approximately 35 files, down from 385.

```
_config.yml                  ~20 lines
Gemfile                      github-pages gem; drop hawkins, jekyll-archives
_layouts/default.html        override: boxed sidebar, portrait, nav, email
assets/css/style.scss        @import the theme, then the cream patch
index.md                     Home
research.md                  Research — the three narrative deep dives
papers.md                    Papers — the list
cv.md                        CV
public-goods.md              Public Goods
404.md
files/cv_letingz.pdf
images/avatar.jpg
images/trendOfBreachAndHIE1.png
images/BBP.PNG
images/itaccess_syn.png
docs/superpowers/specs/      this document
```

### The CSS patch

`assets/css/style.scss` imports `jekyll-theme-minimal`, then overrides only:

| Token | Stock Minimal | This site |
|---|---|---|
| ground | `#ffffff` | `#faf8f5` |
| body text | `#727272` | `#6f6a63` |
| rules | `#e5e5e5` | `#e3ded6` |
| sidebar panel | none | `#f5f2ed`, 8px radius, 1.5rem padding |
| small text | `#777777` | `#7b756c` |

Unchanged: `#222` headings, `#267CB9` links, Noto Sans 14px/1.5, the 860px
wrapper, and the 270px / 500px column split.

The grey shifts are not decoration — stock cool greys on a warm ground read as an
error rather than a choice.

Sidebar nav links: 13px, uppercase, `0.1em` tracking, 9px vertical padding.

### The layout override

Minimal's `_layouts/default.html` renders a sidebar of `site.title`,
`site.description`, and GitHub download buttons. The override replaces the
buttons with site navigation and adds the portrait. Nav links are hard-coded —
five entries that change rarely. If they start changing, move them to
`_data/nav.yml` and loop; not worth the indirection now.

## Content plan

### Carries over (verified as the owner's own)

| Source | Destination |
|---|---|
| `_pages/about.md` — bio, four research topics | `index.md` |
| `_research/p1HIEandDataBreach.md` | `research.md` (section) |
| `_research/p2BugBountyProgram.md` | `research.md` (section) |
| `_research/p3TheFutureOfWorkInUS.md` | `research.md` (section) |
| `files/cv_letingz.pdf` | unchanged path |
| `images/avatar.jpg` | unchanged path |
| Three project figures | unchanged paths |

Fix on migration: `about.md` reads "Assitant Professor". Also update the title to
Assistant Professor of Management Information Systems, Alfred Lerner College of
Business & Economics, University of Delaware.

### Deleted

All 365 remaining tracked files: `_publications/` (5), `_posts/` (25),
`_talks/` (4), `_miscellaneous/` (2), `_drafts/` (1), `_sass/` (112),
`_includes/` (43), `_layouts/` (7), `_data/` (12), `markdown_generator/` (9),
`talkmap/` (7), 23 demo pages under `_pages/`, and the unused portion of
`assets/` + `images/` (73 combined, minus the four keepers).

The personal photographs (`misc_*.jpg`, `logo-piano.jpg`, `Gould.jpg`) are kept
on disk but unreferenced. Decided 2026-08-30: retain the files, build no page
for them now.

### Written from scratch

- **Papers** — currently someone else's list. Ships listing the three working
  papers only. **No placeholder text, no TODO markers, and no empty headings
  reach the published site**; a section with no content is omitted entirely, so
  the page reads as finished rather than unfinished. The published-work section
  appears when its content exists.
- **Public Goods** — currently `# Resource 1` / `# Resource 2`.
- **CV** — the PDF is four years old.

## Branch strategy and rollback

1. `git tag academicpages-final && git push origin academicpages-final` before
   any change. Permanent restore point, independent of branches.
2. All work on branch `minimal-rebuild`. `master` untouched, so the live site
   serves the current content throughout.
3. Preview locally if the Ruby toolchain cooperates; otherwise review the diff
   and merge, since the deployment mechanism itself does not change.
4. Merge to `master` when approved.

Rollback is `git reset --hard academicpages-final` on `master`. Because the Pages
source, branch, and build mechanism are all unchanged, there is no settings
change to reverse — which is one advantage this path has over the al-folio route
that was considered.

**Caveat that rollback does not cover:** permalinks change. The current
`/publication/...`, `/talks/...`, and `/portfolio/` URLs stop existing. Inbound
links from a Scholar profile, a syllabus, or a bookmark would 404. Given those
pages describe another researcher's work, preserving them is not desirable;
`jekyll-redirect-from` is available if any specific URL turns out to matter.

## Verification

Before merge:

- `bundle exec jekyll build` completes without error (or the GitHub Pages build
  succeeds on the branch).
- All four pages render; nav highlights the current page.
- No internal link 404s.
- The CV PDF downloads.
- The portrait loads in the sidebar; the three project figures load on their
  pages.
- Page source contains no reference to `lilianyou.github.io`, `Y. Cheng`,
  `Department of Testing`, or `blog-post-1`.
- Rendered ground is `#faf8f5` and the sidebar panel `#f5f2ed`.

## Resolved 2026-08-30

1. **Research narratives** — get their own `research.md` page, in the nav. The
   split agreed earlier holds: Research carries the deep dives, Papers carries
   the list.
2. **Publication list** — implementation invents nothing. The Papers page ships
   with the three working papers; sections without content are omitted rather
   than stubbed, so the published page never displays placeholder text. The
   published-work list is added when the owner supplies it.
3. **Portrait** — stays under the name in the sidebar.
4. **Personal photographs** — files retained, no page built.

## Still open

- **Source for the published-work list.** Needs a current CV or a dictated
  list. Not a blocker: the site ships correct-but-shorter until then.
- **`.DS_Store` is tracked** and shows as modified. Suggest adding to
  `.gitignore` and untracking during the rebuild.
