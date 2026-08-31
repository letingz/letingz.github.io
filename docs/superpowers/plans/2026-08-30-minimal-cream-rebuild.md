# Minimal + Cream Site Rebuild — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace a 385-file Academic Pages fork — most of whose content belongs to another researcher — with a four-page site on `pages-themes/minimal`, restyled onto paulgp.com's cream ground with a boxed sidebar.

**Architecture:** GitHub Pages builds the site natively from `master` using the supported `jekyll-theme-minimal` gem theme. The custom layer is exactly three files: one `_config.yml`, one overridden `_layouts/default.html`, and one `assets/css/style.scss` that imports the theme and then overrides five colour tokens plus the sidebar box. All content is hand-written Markdown. No collections, no plugins beyond what GitHub Pages already whitelists, no JavaScript.

**Tech Stack:** Jekyll (via the `github-pages` gem), `jekyll-theme-minimal`, SCSS, Markdown, GitHub Pages native build. Local previews run in a `ruby:3.3` Docker container.

**Spec:** `docs/superpowers/specs/2026-08-30-minimal-cream-rebuild-design.md`

## Global Constraints

- **Ground colour is `#faf8f5`**, sidebar panel `#f5f2ed`, body text `#6f6a63`, rules `#e3ded6`, small text `#7b756c`. Headings stay Minimal's `#222`; links stay Minimal's `#267CB9`.
- **Typography is Minimal's own** — Noto Sans 14px/1.5. Do not import Spectral, DM Sans, or any other face.
- **Sidebar box:** `#f5f2ed`, `border-radius: 8px`, `padding: 1.5rem`, `box-sizing: border-box`.
- **Sidebar nav links:** 13px, uppercase, `0.1em` letter-spacing, 9px vertical padding.
- **Portrait sits under the name**, above the description.
- **Four pages only:** Home, Research, CV, Public Goods. No Papers page, no blog, no talks, no teaching, no portfolio.
- **No placeholder text ships.** No "TODO", no "coming soon", no empty headings. A section with no content is omitted from the page entirely.
- **Invent nothing about the publication record.** Only the three projects already in `_research/` may be described. No venues, no co-author names, no dates beyond what the repo already states.
- **Single light theme.** No dark mode, no `prefers-color-scheme` blocks.
- `master` stays deployable and untouched until Task 8.

## Verification approach

This is a static site with no unit-test framework, so "run the tests" means **build the site and assert on the generated output**. Each task states an exact command and its expected result. Two commands recur:

```bash
# BUILD — used in every task's verification
docker run --rm -v "$PWD":/site -w /site -v letingz_gems:/usr/local/bundle \
  ruby:3.3 bash -lc "bundle install --quiet && bundle exec jekyll build"

# SERVE — for eyeballing at http://localhost:4000
docker run --rm -it -v "$PWD":/site -w /site -v letingz_gems:/usr/local/bundle \
  -p 4000:4000 ruby:3.3 bash -lc "bundle install --quiet && bundle exec jekyll serve --host 0.0.0.0"
```

The named volume `letingz_gems` caches installed gems, so only the first run is slow.

**Prerequisite:** Docker Desktop must be running. `docker info` currently fails with `daemon not running`. Start Docker Desktop before Task 2.

---

### Task 1: Safety net and workspace

Creates the permanent restore point and the working branch. No site changes. `master` remains exactly as it is.

**Files:**
- Create: `.gitignore` (or modify if one exists)
- Modify: git refs only

**Interfaces:**
- Consumes: nothing
- Produces: tag `academicpages-final`, branch `minimal-rebuild`. Every later task commits on that branch.

- [ ] **Step 1: Confirm the working tree is clean and you are on master**

```bash
git status --short
git rev-parse --abbrev-ref HEAD
```

Expected: `master`. The only entry in `git status --short` should be ` M .DS_Store` — Step 3 removes it. If anything else is modified, stop and ask before continuing.

- [ ] **Step 2: Create and push the restore tag**

```bash
git tag academicpages-final
git push origin academicpages-final
git tag -l academicpages-final
```

Expected: prints `academicpages-final`. This tag is the rollback target for the whole project and must exist before any file is deleted.

- [ ] **Step 3: Stop tracking .DS_Store**

```bash
cat >> .gitignore <<'EOF'
.DS_Store
_site/
.jekyll-cache/
.jekyll-metadata
vendor/
EOF

git rm --cached -q .DS_Store
git rm --cached -q files/.DS_Store 2>/dev/null || true
```

- [ ] **Step 4: Create the working branch**

```bash
git checkout -b minimal-rebuild
```

- [ ] **Step 5: Verify master is untouched and still deployable**

```bash
git diff master --stat -- . ':!.gitignore' ':!*.DS_Store'
```

Expected: empty output. Nothing but ignore rules differs from `master` yet.

- [ ] **Step 6: Commit**

```bash
git add .gitignore
git commit -m "chore: add .gitignore, untrack .DS_Store, tag pre-rebuild state"
```

---

### Task 2: Strip the fork and stand up bare Minimal

Deletes all inherited content and replaces the build with a minimal, buildable Jekyll site. At the end of this task the site is one page, correctly themed but unstyled beyond stock Minimal.

**Nothing deleted here is lost.** Every file remains in git history and at the `academicpages-final` tag, and the content that survives is reproduced verbatim in Tasks 5–7 of this plan.

**Files:**
- Delete: `_posts/`, `_publications/`, `_talks/`, `_miscellaneous/`, `_drafts/`, `_research/`, `_resources/`, `_sass/`, `_includes/`, `_layouts/`, `_data/`, `_pages/`, `assets/`, `archive/`, `archive_teaching/`, `past/`, `markdown_generator/`, `talkmap/`, `talkmap.ipynb`, `talkmap.py`, `notes.md`, `package.json`, `CHANGELOG.md`, `CONTRIBUTING.md`, `Gemfile.lock`, `.ruby-version`, `files/archive/`, `files/paper1.pdf`, `images/archive/`, `images/safari-pinned-tab.svg`
- Create: `_config.yml`, `Gemfile`, `index.md`
- Keep: `files/cv_letingz.pdf`; `images/avatar.jpg`, `images/BBP.PNG`, `images/itaccess_syn.png`, `images/trendOfBreachAndHIE1.png`; the personal photographs `images/misc_*.jpg`, `images/logo-piano.jpg`, `images/Gould.jpg` (retained unreferenced, per spec decision 5); `LICENSE`, `README.md`, `docs/`

**Interfaces:**
- Consumes: branch `minimal-rebuild` from Task 1
- Produces: a buildable site. `_config.yml` defines `site.title`, `site.description`, and `site.author.email`, which Task 3's layout reads. `Gemfile` pins `github-pages`, which every later build depends on.

- [ ] **Step 1: Confirm Docker is running**

```bash
docker info --format '{{.ServerVersion}}'
```

Expected: a version number. If it prints `failed to connect to the docker API`, start Docker Desktop and wait for it to report ready before continuing.

- [ ] **Step 2: Record the pre-deletion file count**

```bash
git ls-files | wc -l
```

Expected: `385`. Write this number down; Step 9 checks the result against it.

- [ ] **Step 3: Delete the inherited directories and files**

```bash
git rm -rq _posts _publications _talks _miscellaneous _drafts _research _resources \
           _sass _includes _layouts _data _pages assets \
           archive archive_teaching past markdown_generator talkmap

git rm -q talkmap.ipynb talkmap.py notes.md package.json CHANGELOG.md CONTRIBUTING.md \
          Gemfile.lock .ruby-version

git rm -rq files/archive
git rm -q files/paper1.pdf
```

- [ ] **Step 4: Prune images, keeping the four in use and the personal photographs**

Per spec decision 5, the theatre and music photographs are retained on disk even
though no page references them. Only theme furniture is removed.

```bash
cd images
for f in $(git ls-files .); do
  case "$(basename "$f")" in
    # in use by the new site
    avatar.jpg|BBP.PNG|itaccess_syn.png|trendOfBreachAndHIE1.png) ;;
    # retained, unreferenced, per spec decision 5
    misc_*.jpg|logo-piano.jpg|Gould.jpg) ;;
    # theme furniture — goes
    *) git rm -q "$f" ;;
  esac
done
cd ..
git ls-files images
```

Expected: the four in-use images plus `Gould.jpg`, `logo-piano.jpg`, and the four
`misc_*.jpg` files. `safari-pinned-tab.svg` and the `images/archive/` tree should
be gone.

- [ ] **Step 5: Write the new `_config.yml`**

```yaml
title: Leting Zhang
description: >-
  Assistant Professor of Management Information Systems,
  Alfred Lerner College of Business & Economics, University of Delaware
url: "https://letingz.github.io"
baseurl: ""
lang: en-US

theme: jekyll-theme-minimal

author:
  name: Leting Zhang
  email: letingz@udel.edu

plugins:
  - jekyll-seo-tag
  - jekyll-sitemap

exclude:
  - Gemfile
  - Gemfile.lock
  - README.md
  - LICENSE
  - docs/
  - vendor/
```

- [ ] **Step 6: Write the new `Gemfile`**

```ruby
source "https://rubygems.org"

# GitHub Pages builds this site natively; this gem pins the same versions
# GitHub uses, so a local build matches production.
gem "github-pages", group: :jekyll_plugins

# Ruby 3.x no longer ships webrick, which `jekyll serve` needs locally.
gem "webrick", "~> 1.8"
```

- [ ] **Step 7: Write a temporary `index.md`**

Task 5 replaces this with the real Home page. It exists now only so the build has something to render.

```markdown
---
layout: default
title: Home
---

## About

Placeholder — replaced in Task 5.
```

- [ ] **Step 8: Build**

Run the BUILD command from the Verification approach section above.

Expected: exits 0, ending with a `done in N seconds` line. If it fails on `Gemfile.lock` conflicts, confirm Step 3 deleted the old lockfile.

- [ ] **Step 9: Verify the build output and that the fork is gone**

```bash
test -f _site/index.html && echo "index built"
grep -c "Leting Zhang" _site/index.html
grep -rl "lilianyou\|Department of Testing\|blog-post-1\|Y. Cheng" _site/ || echo "no inherited content"
git ls-files | wc -l
```

Expected: `index built`; a `grep -c` result of at least 1; `no inherited content`; and a file count in the mid-30s — down from the 385 recorded in Step 2.

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "feat: strip Academic Pages fork, stand up bare jekyll-theme-minimal

Removes 365 files inherited from the fork, including another researcher's
publications, 24 placeholder posts, and the Academic Pages demo pages.
Replaces the build with the GitHub Pages supported Minimal theme."
```

---

### Task 3: Sidebar layout override

Overrides Minimal's default layout so the sidebar carries the portrait and site navigation instead of the theme's GitHub download buttons.

**Files:**
- Create: `_layouts/default.html`

**Interfaces:**
- Consumes: `site.title`, `site.description`, `site.author.email` from Task 2's `_config.yml`; `images/avatar.jpg`
- Produces: markup hooks that Task 4's stylesheet targets — `header`, `img.portrait`, `nav.sidenav`, and `a[aria-current="page"]` on the active nav link. Do not rename these; Task 4 depends on them exactly.

- [ ] **Step 1: Confirm the theme's stock layout is what we are replacing**

```bash
docker run --rm -v "$PWD":/site -w /site -v letingz_gems:/usr/local/bundle \
  ruby:3.3 bash -lc "bundle exec ruby -e 'puts Gem.loaded_specs[\"jekyll-theme-minimal\"].full_gem_path'"
```

Expected: a gem path. The stock layout lives at `<that path>/_layouts/default.html` and renders `ul.downloads`. Our override replaces it wholesale.

- [ ] **Step 2: Verify the current build has no sidebar navigation**

```bash
grep -c "sidenav" _site/index.html || echo "0 — as expected before the override"
```

Expected: `0 — as expected before the override`.

- [ ] **Step 3: Write `_layouts/default.html`**

```html
<!DOCTYPE html>
<html lang="{{ site.lang | default: "en-US" }}">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    {% seo %}
    <link rel="stylesheet" href="{{ "/assets/css/style.css" | relative_url }}">
  </head>
  <body>
    <div class="wrapper">
      <header>
        <h1><a href="{{ "/" | relative_url }}">{{ site.title }}</a></h1>

        <img class="portrait"
             src="{{ "/images/avatar.jpg" | relative_url }}"
             alt="{{ site.author.name }}">

        <p>{{ site.description }}</p>

        <nav class="sidenav">
          {% assign here = page.url | remove: "index.html" %}
          <a href="{{ "/" | relative_url }}"
             {% if here == "/" %}aria-current="page"{% endif %}>Home</a>
          <a href="{{ "/research/" | relative_url }}"
             {% if here contains "/research" %}aria-current="page"{% endif %}>Research</a>
          <a href="{{ "/cv/" | relative_url }}"
             {% if here contains "/cv" %}aria-current="page"{% endif %}>CV</a>
          <a href="{{ "/public-goods/" | relative_url }}"
             {% if here contains "/public-goods" %}aria-current="page"{% endif %}>Public Goods</a>
        </nav>

        <p><small>{{ site.author.email }}</small></p>
      </header>

      <section>
        {{ content }}
      </section>

      <footer>
        <p><small>&copy; {{ site.time | date: "%Y" }} {{ site.author.name }}</small></p>
      </footer>
    </div>
  </body>
</html>
```

- [ ] **Step 4: Rebuild**

Run the BUILD command.

Expected: exits 0.

- [ ] **Step 5: Verify the sidebar renders with all four links and the portrait**

```bash
grep -c 'class="sidenav"' _site/index.html
grep -o '>\(Home\|Research\|CV\|Public Goods\)<' _site/index.html | sort -u
grep -c 'class="portrait"' _site/index.html
grep -c 'aria-current="page"' _site/index.html
grep -c "downloads" _site/index.html || echo "download buttons gone"
```

Expected: `1` for sidenav; all four link labels listed; `1` for portrait; `1` for `aria-current` (Home is the current page); `download buttons gone`.

- [ ] **Step 6: Commit**

```bash
git add _layouts/default.html
git commit -m "feat: override Minimal layout with portrait and site navigation"
```

---

### Task 4: Cream stylesheet

Adds the only stylesheet in the project. Imports the theme, then overrides five colour tokens and boxes the sidebar.

**Files:**
- Create: `assets/css/style.scss`

**Interfaces:**
- Consumes: the markup hooks from Task 3 — `header`, `.portrait`, `.sidenav`, `a[aria-current="page"]`
- Produces: `/assets/css/style.css` in the built site, which Task 3's layout already links

- [ ] **Step 1: Verify the ground is currently stock white**

```bash
grep -o "background-color:#fff\|background-color: #fff" _site/assets/css/style.css | head -1
```

Expected: a match on stock white. This is the state Step 2 changes. If the file does not exist yet, that is also expected — the theme only emits it once a source `style.scss` exists.

- [ ] **Step 2: Write `assets/css/style.scss`**

The empty front matter block at the top is required — it tells Jekyll to process the file through SCSS rather than copy it verbatim.

```scss
---
---

@import "{{ site.theme }}";

// ---------------------------------------------------------------------------
// Cream patch over pages-themes/minimal.
// Ground and sidebar panel are paulgp.com's; everything else is stock Minimal.
// The grey shifts are not decoration: stock cool greys on a warm ground read
// as a mistake rather than a choice.
// ---------------------------------------------------------------------------

$ground:      #faf8f5;
$body-text:   #6f6a63;
$rule:        #e3ded6;
$panel:       #f5f2ed;
$small-text:  #7b756c;
$heading:     #222222;
$link:        #267cb9;

body {
  background-color: $ground;
  color: $body-text;
}

small { color: $small-text; }
hr { background: $rule; }

// --- sidebar as a paulgp-style box -----------------------------------------

header {
  box-sizing: border-box;
  background: $panel;
  border-radius: 8px;
  padding: 1.5rem;
}

header h1 {
  font-size: 24px;
  margin-bottom: 14px;
}

header h1 a { color: $heading; }
header h1 a:hover { color: $heading; font-weight: inherit; }

header > p {
  font-size: 13px;
  line-height: 1.55;
  margin-bottom: 18px;
}

.portrait {
  display: block;
  width: 100%;
  max-width: 180px;
  aspect-ratio: 1 / 1;
  border-radius: 50%;
  object-fit: cover;
  margin: 0 auto 18px;
}

.sidenav {
  border-top: 1px solid $rule;
  padding-top: 10px;
  margin-bottom: 14px;
}

.sidenav a {
  display: block;
  font-size: 13px;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: $small-text;
  padding: 9px 0;
  line-height: 1.5;
}

// Minimal bolds links on hover, which makes nav items jitter. Suppress it here.
.sidenav a:hover {
  color: $heading;
  font-weight: 400;
}

.sidenav a[aria-current="page"] { color: $link; }

// --- content ---------------------------------------------------------------

.figure {
  background: $panel;
  border: 1px solid $rule;
  border-radius: 4px;
  padding: 12px;
  margin: 0 0 12px;
}

.figure img { display: block; width: 100%; }

.project { margin-bottom: 32px; }

footer {
  border-top: 1px solid $rule;
  padding-top: 16px;
}

// --- narrow screens --------------------------------------------------------
// Minimal unfloats its columns below 960px; keep the box readable there.

@media print, screen and (max-width: 960px) {
  header {
    position: static;
    float: none;
    width: auto;
    margin-bottom: 24px;
  }
  .portrait { max-width: 150px; }
}
```

- [ ] **Step 3: Rebuild**

Run the BUILD command.

Expected: exits 0. An SCSS syntax error fails the build loudly; if it does, the error names the line.

- [ ] **Step 4: Verify the compiled CSS carries the cream tokens**

```bash
grep -c "faf8f5" _site/assets/css/style.css
grep -c "f5f2ed" _site/assets/css/style.css
grep -c "6f6a63" _site/assets/css/style.css
grep -c "border-radius:8px\|border-radius: 8px" _site/assets/css/style.css
```

Expected: each returns at least `1`.

- [ ] **Step 5: Look at it**

Run the SERVE command, open `http://localhost:4000`, and confirm against the approved mockup at
https://claude.ai/code/artifact/be74b4b6-1bd9-4fc7-87a4-22e6345d14ac —
cream ground, sidebar in a rounded panel, round portrait under the name, four uppercase nav links. Stop the server with Ctrl-C.

- [ ] **Step 6: Commit**

```bash
git add assets/css/style.scss
git commit -m "feat: cream ground and boxed sidebar over Minimal"
```

---

### Task 5: Home page

Replaces the Task 2 placeholder with the real Home page. Content comes from the old `_pages/about.md`, corrected and updated.

**Files:**
- Modify: `index.md` (full rewrite)

**Interfaces:**
- Consumes: `_layouts/default.html` from Task 3
- Produces: the site's root page; links to `/research/`, which Task 6 creates

- [ ] **Step 1: Verify the placeholder is still in place**

```bash
grep -c "Placeholder — replaced in Task 5" index.md
```

Expected: `1`.

- [ ] **Step 2: Write `index.md`**

Two corrections to the source material are deliberate: the old page read "Assitant Professor", and it did not name the college.

```markdown
---
layout: default
title: Home
---

## About

I am an Assistant Professor of Management Information Systems at the Alfred Lerner
College of Business & Economics, University of Delaware. I received my Ph.D. in
Management Information Systems from the Fox School of Business, Temple University.

My research examines how organizations manage the risks and returns of information
technology, spanning information security, health information systems, and the
effects of information technology on labor markets.

## Research Interests

- Information technology risks
- Health information technology
- Labor market
- Digital innovation

## Current Work

I have three projects in progress: on health information exchange and data breaches,
on bug bounty program design, and on access to IT resources and regional
unemployment. [Read about them here.](/research/)

## Contact

- letingz@udel.edu
- Alfred Lerner College of Business & Economics, University of Delaware, Newark, DE
```

- [ ] **Step 3: Rebuild**

Run the BUILD command. Expected: exits 0.

- [ ] **Step 4: Verify the page renders correctly and the typo is gone**

```bash
grep -c "Assistant Professor" _site/index.html
grep -c "Assitant" _site/index.html || echo "typo fixed"
grep -c "Placeholder" _site/index.html || echo "placeholder gone"
grep -o 'href="/research/"' _site/index.html | head -1
```

Expected: at least `1` for "Assistant Professor"; `typo fixed`; `placeholder gone`; and the research link present.

- [ ] **Step 5: Commit**

```bash
git add index.md
git commit -m "content: write Home page, fix Assitant typo, name the college"
```

---

### Task 6: Research page

The largest content task. Carries the three project narratives from the deleted `_research/` files, reproduced here in full so no earlier state needs consulting.

**Files:**
- Create: `research.md`

**Interfaces:**
- Consumes: `_layouts/default.html`; `images/trendOfBreachAndHIE1.png`, `images/BBP.PNG`, `images/itaccess_syn.png`; the `.project` and `.figure` classes from Task 4
- Produces: the page at `/research/` that Task 5's Home page links to

- [ ] **Step 1: Confirm the three figures survived Task 2**

```bash
ls images/trendOfBreachAndHIE1.png images/BBP.PNG images/itaccess_syn.png
```

Expected: all three listed. If any is missing, recover it with
`git checkout academicpages-final -- images/<name>` before continuing.

- [ ] **Step 2: Write `research.md`**

Every factual claim below is taken verbatim from the deleted `_research/` files. Do not add venues, dates, co-authors, or publication status — none is known.

```markdown
---
layout: default
title: Research
permalink: /research/
---

## Research

Longer write-ups of the projects I am working on.

<div class="project" markdown="1">

### Does Sharing Make My Data More Insecure? Health Information Exchange and Data Breaches

<div class="figure"><img src="/images/trendOfBreachAndHIE1.png" alt="Trend of data breaches and health information exchange participation"></div>

This paper examines the information security implications of participating in
inter-organizational systems in the context of the healthcare industry. Public
concern regarding data breach risks has increased as more hospitals share their
data through electronic Health Information Exchange (HIE) systems.

To study the impact of joining an HIE on a hospital's data breach risk, we use six
years of panel data on hospital characteristics, HIE participation status, and data
breach incidents from multiple sources.

The results show that the likelihood a hospital experiences a data breach decreases
by 1.7 percentage points — a 43% reduction — after joining an HIE. The magnitude of
that reduction is larger for HIE member hospitals with a higher ex-ante IT security
investment level. The likelihood of breaches caused by insiders or illegal access to
IT systems decreases significantly after a hospital joins an HIE, whereas there is
no significant impact on breaches caused by outsiders or on physical breaches.

</div>

<div class="project" markdown="1">

### How to Make My Bug Bounty Cost-effective? A Game-theoretical Model

<div class="figure"><img src="/images/BBP.PNG" alt="Bug bounty program model"></div>

To mitigate the threats from malicious exploitation of vulnerabilities, an
increasing number of organizations across different industries have started
incorporating bug bounty programs into their vulnerability management cycles. A bug
bounty program attracts a crowd of external security researchers to search for new
vulnerabilities in an organization's IT systems.

As bug bounty programs have gained prevalence, it has become important for an
organization to understand how the characteristics of a crowd affect its own
response and the performance of its program from a cost-effectiveness perspective.

We use a game-theoretical model to examine these questions, considering the
efficiency of security researchers in discovering vulnerabilities, the heterogeneity
among those researchers, and their number.

</div>

<div class="project" markdown="1">

### Does Access to Information Technology Resources Reduce Rise in Regional Unemployment Amidst Global Disasters? Insights from COVID-19 in the United States

<div class="figure"><img src="/images/itaccess_syn.png" alt="IT access and regional unemployment"></div>

One major change in working conditions after the pandemic is the increased
prevalence of transformations geared towards facilitating working from home. Firms
have to allocate new, or reallocate existing, digital resources to support the
restructuring that working from home requires. It is not clear to what extent access
to internal and external IT resources reduces those frictions, nor which types of IT
resources help most.

Using the introduction of stay-at-home orders across 48 states during the first wave
of COVID-19, together with natural variation in the availability of IT services and
resources across counties, we use a difference-in-differences specification to
examine the frictions of transitioning to working from home. The identification
strategy rests on the premise that businesses in regions without adequate IT
resource access lack the means to transform jobs, so at least a portion of
non-transformable positions must be eliminated.

Our findings show that counties with adequate business access to IT resources
experienced lower rates of monthly unemployment after stay-at-home orders were
enforced. Examining which resources mattered, investments in DevOps, work-from-home
accessibility products, and network infrastructure appear to have smoothed some of
the transition frictions, while investments in enterprise software, database
infrastructure, and cybersecurity products may have exacerbated them.

</div>
```

- [ ] **Step 3: Rebuild**

Run the BUILD command. Expected: exits 0.

- [ ] **Step 4: Verify the page renders with all three projects and their figures**

```bash
test -f _site/research/index.html && echo "research page built"
grep -c "class=\"project\"" _site/research/index.html
grep -o 'src="/images/[a-zA-Z0-9._]*"' _site/research/index.html | sort -u
grep -c "1.7 percentage points" _site/research/index.html
grep -c "TODO\|placeholder\|Coming soon" _site/research/index.html || echo "no placeholder text"
```

Expected: `research page built`; `3` projects; the three image paths listed; at least `1` for the headline finding; `no placeholder text`.

- [ ] **Step 5: Verify the images actually resolve**

```bash
for f in trendOfBreachAndHIE1.png BBP.PNG itaccess_syn.png; do
  test -f "_site/images/$f" && echo "ok $f" || echo "MISSING $f"
done
```

Expected: three `ok` lines. A `MISSING` line means the file was excluded from the build — check `_config.yml`'s `exclude` list.

- [ ] **Step 6: Commit**

```bash
git add research.md
git commit -m "content: write Research page with the three project narratives"
```

---

### Task 7: CV and Public Goods pages

The two remaining pages. Both are short; they share a task because neither warrants its own review gate.

**Files:**
- Create: `cv.md`, `public-goods.md`, `404.md`

**Interfaces:**
- Consumes: `_layouts/default.html`; `files/cv_letingz.pdf`
- Produces: the pages at `/cv/` and `/public-goods/` that Task 3's navigation links to

- [ ] **Step 1: Confirm the CV PDF survived Task 2**

```bash
ls -la files/cv_letingz.pdf
```

Expected: the file exists, about 98KB. Note for the owner: it dates to August 2022 and is the oldest content on the site.

- [ ] **Step 2: Write `cv.md`**

The old page embedded the PDF in a fixed-size `<iframe>` that overflowed on narrow screens. A download link is better behaved and works on mobile.

```markdown
---
layout: default
title: CV
permalink: /cv/
---

## Curriculum Vitae

[Download my CV (PDF)](/files/cv_letingz.pdf)

### Appointments

- Assistant Professor, Management Information Systems — Alfred Lerner College of
  Business & Economics, University of Delaware

### Education

- Ph.D., Management Information Systems — Fox School of Business, Temple University
```

- [ ] **Step 3: Write `public-goods.md`**

Per the global constraints, this page ships with a real sentence rather than a placeholder. It describes what the page is for; the owner adds entries when there are entries to add.

```markdown
---
layout: default
title: Public Goods
permalink: /public-goods/
---

## Public Goods

Data, code, and teaching materials I share with the field.

If you are looking for materials related to one of my projects, please
[email me](mailto:letingz@udel.edu) — I am glad to share.
```

- [ ] **Step 4: Write `404.md`**

```markdown
---
layout: default
title: Page not found
permalink: /404.html
---

## Page not found

That page does not exist. Try the [home page](/), or
[email me](mailto:letingz@udel.edu) if you were looking for something specific.
```

- [ ] **Step 5: Rebuild**

Run the BUILD command. Expected: exits 0.

- [ ] **Step 6: Verify both pages build and the CV downloads**

```bash
test -f _site/cv/index.html && echo "cv built"
test -f _site/public-goods/index.html && echo "public goods built"
test -f _site/404.html && echo "404 built"
test -f _site/files/cv_letingz.pdf && echo "cv pdf copied"
grep -c "TODO\|Resource 1\|Coming soon" _site/cv/index.html _site/public-goods/index.html || echo "no placeholder text"
```

Expected: four confirmation lines, then `no placeholder text`.

- [ ] **Step 7: Commit**

```bash
git add cv.md public-goods.md 404.md
git commit -m "content: write CV, Public Goods, and 404 pages"
```

---

### Task 8: Full verification and merge

Runs the spec's acceptance checklist against the built site, then merges to `master`. This is the only task that changes what the public sees.

**Files:**
- Modify: `README.md`
- Modify: git refs (merge to `master`)

**Interfaces:**
- Consumes: everything from Tasks 1–7
- Produces: the live site

- [ ] **Step 1: Clean rebuild from scratch**

```bash
rm -rf _site .jekyll-cache
docker run --rm -v "$PWD":/site -w /site -v letingz_gems:/usr/local/bundle \
  ruby:3.3 bash -lc "bundle install --quiet && bundle exec jekyll build"
```

Expected: exits 0. A clean build catches anything that only worked because of stale cache.

- [ ] **Step 2: Run the spec's acceptance checklist**

```bash
echo "--- all four pages exist ---"
for p in index.html research/index.html cv/index.html public-goods/index.html; do
  test -f "_site/$p" && echo "ok $p" || echo "MISSING $p"
done

echo "--- no inherited content anywhere ---"
grep -rl "lilianyou\|Y\. Cheng\|Department of Testing\|blog-post-1\|Resource 1" _site/ \
  || echo "clean"

echo "--- no placeholder text anywhere ---"
grep -rl "TODO\|Lorem ipsum\|Coming soon\|Placeholder" _site/ || echo "clean"

echo "--- palette ---"
grep -c "faf8f5" _site/assets/css/style.css
grep -c "f5f2ed" _site/assets/css/style.css

echo "--- assets resolve ---"
for f in images/avatar.jpg images/BBP.PNG images/itaccess_syn.png \
         images/trendOfBreachAndHIE1.png files/cv_letingz.pdf; do
  test -f "_site/$f" && echo "ok $f" || echo "MISSING $f"
done

echo "--- nav on every page ---"
for p in index.html research/index.html cv/index.html public-goods/index.html; do
  printf "%s: %s nav links\n" "$p" "$(grep -o 'class="sidenav"' "_site/$p" | wc -l | tr -d ' ')"
done
```

Expected: four `ok` page lines; `clean` twice; both palette greps at least `1`; five `ok` asset lines; and every page reporting `1` sidenav.

- [ ] **Step 3: Check every internal link resolves**

```bash
grep -rho 'href="/[^"]*"' _site/ | sort -u | sed 's/href="//;s/"//' | while read -r u; do
  case "$u" in
    */) t="_site${u}index.html" ;;
    *)  t="_site${u}" ;;
  esac
  test -e "$t" || echo "BROKEN: $u"
done
echo "link check done"
```

Expected: `link check done` with no `BROKEN` lines above it.

- [ ] **Step 4: Final visual check**

Run the SERVE command and walk all four pages at `http://localhost:4000`. Confirm against
https://claude.ai/code/artifact/be74b4b6-1bd9-4fc7-87a4-22e6345d14ac —
cream ground, boxed sidebar, portrait under the name, nav highlighting the current page, three projects with figures on Research. Ctrl-C to stop.

- [ ] **Step 5: Rewrite `README.md`**

The current README describes Academic Pages and its fork instructions, none of which now apply.

```markdown
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
```

- [ ] **Step 6: Commit the README**

```bash
git add README.md
git commit -m "docs: rewrite README for the Minimal-based site"
```

- [ ] **Step 7: Confirm the restore point still exists before merging**

```bash
git tag -l academicpages-final
git rev-parse academicpages-final
```

Expected: the tag name and a commit SHA. **Do not proceed if this is empty** — it is the only rollback path.

- [ ] **Step 8: Merge to master**

```bash
git checkout master
git merge --no-ff minimal-rebuild -m "feat: rebuild site on Minimal with cream palette

Replaces the Academic Pages fork with a four-page site: Home, Research, CV,
Public Goods. Removes 365 inherited files, including another researcher's
publication list and 24 placeholder posts.

Pre-rebuild state preserved at tag academicpages-final."
git push origin master
```

- [ ] **Step 9: Confirm the live site**

Wait two to three minutes for the GitHub Pages build, then check the repository's Actions or Pages settings for a green build, and open:

- https://letingz.github.io/
- https://letingz.github.io/research/
- https://letingz.github.io/cv/
- https://letingz.github.io/public-goods/

Expected: all four load with the cream ground and boxed sidebar; the portrait and the three figures display; the CV downloads.

**If the build fails or the site looks wrong:**

```bash
git reset --hard academicpages-final
git push --force-with-lease origin master
```

The previous site returns on the next Pages build. Because the Pages source, branch, and build mechanism never changed, there is no settings change to undo.

---

## Deferred — not part of this plan

- **The published-work list.** Requires a current CV or a dictated list from the owner. It joins `research.md` as a `## Published` section when supplied. Until then the section does not exist, per the global constraint against empty headings.
- **A replacement CV PDF.** The current file dates to August 2022.
- **A personal page** for the theatre and music photographs. The images stay in `images/` through the rebuild, unreferenced, so building the page later needs no recovery step.
- **Redirects for old URLs.** `/publication/*`, `/talks/*`, and `/portfolio/` will 404. Those pages described another researcher's work, so this is intended. If a specific URL turns out to matter, `jekyll-redirect-from` is on the GitHub Pages whitelist.
