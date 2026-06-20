# guacamayalab.com

Personal notebook/blog. Hugo + a custom theme (`themes/papermod-lite/`)
built specifically for this site. Text and images only — no app-like
features. Optimize for "boring and correct" over "clever."

## Stack

- Hugo (static site, no JS framework, no build step beyond `hugo` itself).
  **Requires Hugo v0.146.0+** — this repo uses the template/layout system
  introduced in that release, not the older `_default`/`partials` one. If
  `hugo version` reports anything older, upgrade before debugging anything
  template-related; an old Hugo binary will fail to find half these files.
- Theme: `themes/papermod-lite/` — vendored directly in this repo, not a
  submodule. Edit it in place; there's no upstream to stay in sync with.
- Deploy: Cloudflare Pages. Build command `hugo`, publish dir `public/`.

## Commands

```
hugo server -D       # local dev, includes drafts
hugo new posts/my-post-title.md   # new post
hugo new about.md                  # new page (see "Pages" below)
hugo                               # production build → public/
```

## Structure

```
content/
  posts/*.md   → blog posts (dated, listed on the homepage)
  *.md         → standalone pages (about, etc.) — optional, see below
  _index.md    → optional intro text shown above the post list on the homepage
themes/papermod-lite/
  layouts/
    baseof.html       → base HTML shell, wraps everything
    home.html          → homepage (post list)
    home.llms.txt      → auto-generated llms.txt (LLMS output format)
    page.html           → generic page template (title + content only)
    list.html            → fallback for tag/term listing pages
    posts/page.html       → post template (date, byline, content, tags)
    _partials/             → header.html, footer.html, head.html
  assets/css/style.css   → the entire stylesheet, inlined at build time
```

Note the current Hugo naming: `home.html` not `index.html`, `_partials/`
not `partials/`, no `_default/` folder at all (those files live at the
`layouts/` root now), and `page.html` is the Page-*kind* template name —
it's not literally "the page content type," every regular content page
(posts included) has Kind `page`. The `posts/page.html` override exists
because it's more specific (matches the `posts` Page Path), so it wins
over the root `page.html` for anything under `content/posts/`.

## Decisions already made — don't re-litigate these without being asked

These look like they could be "improved" or "simplified" but they're
deliberate. If something here seems off, ask before changing it.

**Permalinks are flat: `/post-title/`, not `/2026/04/18/post-title/`.**
No date folders. This is evergreen writing, not a news site — dated URLs
age content visually and add nothing. Slugs come from the filename
directly (no `slug:` front matter field on posts); keep post filenames
lowercase-hyphenated and the URL is already right. Configured in
`hugo.toml` under the current nested-by-page-kind `permalinks.page.posts`
syntax — the old flat `permalinks.posts = "..."` form is no longer how
current Hugo docs show it, don't revert to that shape.

**Homepage/list excerpts use the post's first paragraph, not Hugo's
`.Summary`.** See `layouts/home.html` / `layouts/list.html`:
`{{ index (split .Content "</p>") 0 | plainify }}`. Posts here are written
with a short hook line, then a blank line, then the body — that's a
deliberate structure, not an accident. Word-count truncation (Hugo's
default `.Summary`) ignores that structure and cuts mid-sentence or
mid-word. `summaryLength` in `hugo.toml` is kept around only as a fallback
for the meta-description tag and the RSS feed, where word-count truncation
is fine because paragraph fidelity doesn't matter there.

**One `<h1>` per page.** Site title is `<h1>` only on the homepage (a `<p>`
elsewhere); post/page title is the `<h1>` on its own page. This was a
genuine bug fix (every page used to share one `<h1>`, which is bad for
SEO/AIO), not a style preference — don't merge them back.

**`head.html` delegates Open Graph and Twitter Card tags to Hugo's own
built-in embedded partials** (`{{ partial "opengraph.html" . }}` /
`{{ partial "twitter_cards.html" . }}`) rather than hand-rolled meta tags —
Hugo's versions handle fallback chains, image discovery, and locale more
robustly than a hand-rolled version would, and they're maintained upstream.
Configured via `params.description` (and optionally `params.social.twitter`)
in `hugo.toml`. **Structured data (JSON-LD) is still hand-rolled** — a
deliberately minimal `BlogPosting` block (title, date, author, url, 4
fields) in `head.html`, gated on `{{ if eq .Type "posts" }}`. Hugo also
ships a built-in `schema.html` partial, but it emits *microdata*, not
JSON-LD; JSON-LD is the currently-preferred structured-data format, so
this stays custom rather than switching to the built-in. Don't expand the
JSON-LD block to mirror PaperMod's full schema — keep it to the fields
that actually matter.

**Google Analytics uses Hugo's built-in embedded partial**:
`{{ partial "google_analytics.html" . }}` in `head.html`, configured via
`services.googleAnalytics.id` (lowercase `id`) in `hugo.toml`. The old
`{{ template "_internal/google_analytics.html" . }}` syntax you'll see in
older tutorials/Stack Overflow answers no longer exists — Hugo removed the
`_internal` template namespace entirely in v0.146.0.

**Theme scope is deliberately small.** Three layouts only: home, post,
page. No search, no tags-listing page beyond a plain list, no pagination
UI, no cover images, no social icon row, no comments, no i18n. These were
cut on purpose during a "what do we actually need" pass — don't add any of
them back unless explicitly asked, even if PaperMod (the theme this was
distilled from) has them.

**Light/dark toggle and the anti-flash theme script in `head.html`** are
adapted from hugo-PaperMod (MIT-licensed, credited in
`themes/papermod-lite/LICENSE`). If touching theme switching, that's the
reference implementation to check against — don't reinvent it.

**Templates use the global `site` function, not `.Site`.** E.g.
`site.Params.author`, `site.Title`, `site.Menus.main`, not
`.Site.Params.author`. Several `.Site.X` accessors (`.Site.Data`,
`.Site.Languages`, `.Site.Sites`, `.Site.Author`) have already been
deprecated in favor of global functions (`hugo.Data`, etc.); `.Site.Params`
and friends aren't formally deprecated yet, but current Hugo docs
consistently use the lowercase global `site` function in their own
examples, so that's what this repo follows. Don't reintroduce `.Site.X`.

**Config uses `locale`, not `languageCode`** (the latter was deprecated in
Hugo v0.158.0). Don't switch it back even though plenty of older
tutorials/themes still show `languageCode`.

**`llms.txt` is auto-generated by Hugo, not hand-written.** It's produced
by `layouts/home.llms.txt` via the `LLMS` custom output format defined in
`hugo.toml`. Google's PageSpeed Insights started flagging its absence as an
error in mid-2026 (still labeled experimental on their end). The file
updates automatically with every new post — don't replace it with a static
version in `static/`.

**CSS is inlined at build time, not served as a separate file.** The
stylesheet lives at `themes/papermod-lite/assets/css/style.css` (not
`static/`) so Hugo's resource pipeline can inline it via
`resources.Get "css/style.css" | safeCSS` in `head.html`. This eliminates
the render-blocking external stylesheet request. Don't move it back to
`static/` — that breaks the inlining.

## Pages

A page is anything in `content/` outside `content/posts/` — e.g.
`content/about.md`. It uses `layouts/page.html` (title + content, no
date/byline/tags), not the post template.

To make a page show up in the header nav, add `menu: main` to its front
matter:

```yaml
---
title: "About"
draft: false
menu: main
---
```

No template or config change needed for this — Hugo merges front-matter
menu entries with the `[[menu.main]]` entries in `hugo.toml` into the same
nav automatically. Zero pages → nav is exactly whatever's in
`hugo.toml`'s `[[menu.main]]` (today: just "About" → yatan.co). Add a page
with `menu: main` → it appears alongside that, no other changes required.

## Working style for this repo

- Ultra-simple and robust over clever. If a fix needs more than ~20 lines
  of template logic, stop and ask whether there's a simpler way — there
  usually is.
- Don't add dependencies, build tooling, or JS unless something genuinely
  can't be done without it. This site is static HTML + one CSS file on
  purpose.
- Don't build/propose new features unprompted. If asked to think through
  a problem, that's a request for discussion, not an invitation to start
  writing code.
- Before adding a new theme feature, check the "deliberately small" list
  above — if it's on there, confirm with me first rather than assuming
  the omission was an oversight.
- When in doubt about current Hugo syntax (config keys, template
  functions, layout naming), check https://gohugo.io/documentation/
  rather than relying on training data — Hugo's template system changed
  substantially in v0.146.0 and a lot of search results / cached knowledge
  predate that.
