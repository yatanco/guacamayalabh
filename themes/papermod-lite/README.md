# papermod-lite

A minimal Hugo theme for a personal notebook or small blog. **Requires Hugo
v0.146.0+** (uses the current template/layout system introduced in that
release — see "Hugo version notes" below if migrating an older site).

Three layouts:

1. **Home** (`layouts/home.html`) — lists your posts, newest first.
   Optionally shows intro text above the list if you put content in
   `content/_index.md`.
2. **Post** (`layouts/posts/page.html`, renders `content/posts/*.md`) —
   date, title, byline, content, optional tags, back link.
3. **Page** (`layouts/page.html`, renders any `.md` outside
   `content/posts/`, e.g. `content/about.md`) — just title + content. No
   date or byline, since it's not a blog post.

Light/dark toggle (adapted from hugo-PaperMod, MIT) is on by default. Open
Graph and Twitter Card tags come from Hugo's own built-in embedded
partials. Google Analytics wires in automatically when you set an ID.
That's it.

## Setup

```
themes/
  papermod-lite/   ← this folder
```

```toml
# hugo.toml
theme = "papermod-lite"
```

## Config reference

```toml
locale = "en-US"   # not languageCode — that key was deprecated in Hugo v0.158.0

[params]
  tagline            = "An online notebook — unfinished, evolving, honest."
  description         = "Fallback meta description / Open Graph description."
  author             = "Yatan"
  defaultTheme       = "auto"     # "auto" | "light" | "dark"
  disableThemeToggle = false
  postsSectionTitle  = "Latest Notes"   # heading above the post list
  backLinkText       = "Back"           # link text at the bottom of each post/page

[services.googleAnalytics]
  id = "G-XXXXXXXXXX"   # lowercase `id`; only fires on production builds

[permalinks]
  [permalinks.page]
    posts = "/:slug/"   # flat post URLs — see hugo.toml comments for why

[[menu.main]]
  name = "About"
  url  = "https://yatan.co"
```

## Adding a post

```
hugo new posts/my-post-title.md
```

Set `draft: false` when it's ready.

## Adding a page

```
hugo new about.md
```

This creates `content/about.md` using the Page layout (title + content,
no blog metadata). To make it show up in the header nav automatically,
uncomment one line in the generated front matter:

```yaml
---
title: "About"
draft: false
menu: main
---
```

No template or config edit needed — Hugo merges front-matter menu entries
with the `[[menu.main]]` config entries into the same nav automatically.
If you never create a page, the nav stays exactly as configured (today:
just the "About" link out to yatan.co). Add a page with `menu: main` and
it appears alongside it.

Optional: add `weight: 5` under the page's front matter if you want to
control where it sits relative to other nav items (lower weight = earlier).

## Optional intro block

Create `content/_index.md` with front matter + body text. The body renders
above the post list. Leave the file empty (or don't create it) and the list
starts immediately.

## Hugo version notes

Hugo did a full template-system overhaul in **v0.146.0**: the `_default`
folder was removed (files moved to the `layouts/` root), `layouts/partials`
was renamed to `layouts/_partials`, the homepage template `index.html`
became `home.html`, and `_internal` template calls (e.g.
`{{ template "_internal/google_analytics.html" . }}`) were replaced by
ordinary partial calls (`{{ partial "google_analytics.html" . }}`). This
theme is built against the post-overhaul structure — if you're looking at
an older Hugo theme tutorial or a pre-2025 example online, the file names
and a few config keys (`languageCode` → `locale`, flat `permalinks.posts`
→ nested `permalinks.page.posts`) will look different from what's here.
Both `languageCode`/`locale` keys still parse without error on recent Hugo
versions, but `locale` is what current docs document; same logic for
permalinks. This theme follows the current documented form for both.

## What's not here

Search, tags listing page styling beyond a plain list, pagination UI,
cover images, social icons, comments, i18n, breadcrumbs. Add if a specific
site needs one — but that's a new decision, not a default.
