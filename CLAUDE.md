# jimmerlitter.com — Hugo site notes

Personal site for James Huang. Hugo v0.164 (extended), theme = PaperMod.

## Structure

- `hugo.toml` — site config: profileMode buttons (home page), main menu, taxonomies (`tags`, `categories`), markup/goldmark (KaTeX passthrough for math).
- `themes/PaperMod/` — **git submodule** (`adityatelange/hugo-PaperMod`). Never edit files in here directly — changes live in a different repo and will be lost/conflict on submodule update.
- `layouts/` — project-level template overrides. Hugo resolves a template from the project `layouts/` tree before falling back to the theme's, path-for-path. To customize theme behavior, copy the theme file to the same relative path under project `layouts/` and edit the copy. Current overrides:
  - `layouts/single.html` — full override of theme's single-post template. Diff from stock: post tags render inside `<header class="post-header">` (top of article, right after post-meta) instead of in the `post-footer` at the bottom.
  - `layouts/list.html` — full override of theme's list template (used by `/posts/`, `/books/`, `/tags/<term>/`, etc., but not `.IsHome` when profileMode is on — that goes through `index_profile.html`). Diff from stock: each entry card shows its tags as pills (`<ul class="post-tags entry-tags">`) between the summary and the meta footer.
  - `layouts/athletic/list.html` — section-specific override for `/athletic/`; renders a photo gallery grid from a `photos` front-matter param instead of the normal post-entry list.
  - `layouts/partials/extend_head.html` — loads KaTeX (CSS/JS) only on pages with `math: true` in front matter.
  - `layouts/_partials/extend_footer.html` — JS to force external links (and the resume PDF) to open in a new tab.
  - `layouts/_partials/comments.html` — giscus (GitHub Discussions) comments, config in `hugo.toml [params.giscus]`; syncs light/dark with PaperMod's theme toggle.
  - `assets/css/extended/custom.css` — project CSS, auto-concatenated after the theme's stylesheet (PaperMod convention: anything under `assets/css/extended/*.css` gets picked up automatically, no template wiring needed). Contains: profile image border-radius, athletic gallery grid, and spacing/z-index rules for the moved/added tag pills (list-card tag pills need `position:relative;z-index:1` to stay clickable above the card's full-bleed `.entry-link` overlay).

- `content/`
  - `about/index.md`, `books/_index.md`, `creative/_index.md` (placeholder, no posts yet), `athletic/_index.md` + individual entries — see `layouts/athletic/list.html` for the `photos` front-matter schema.
  - `posts/` — blog. Front matter uses `tags: [...]` and `categories: [...]`. Page bundles (e.g. `posts/blog-001/`) hold their own `figures/` images referenced with relative paths.

## Taxonomies / tags

- Configured in `hugo.toml`: `[taxonomies] category = "categories"`, `tag = "tags"`.
- `/tags/` — auto-generated list of all tags with counts (theme's `layouts/taxonomy.html`, not overridden).
- `/tags/<name>/` — auto-generated list of posts with that tag (uses `layouts/list.html`, i.e. the project override above, so tag pages also show each post's other tags).
- Home page has a "Tags" button in `params.profileMode.buttons` (`hugo.toml`) linking to `tags` — added in place of the old "Creative" button since `content/creative/` had no real content yet. Profile buttons are plain relative hrefs (no leading slash), which only works because they're rendered on the home page (root URL) via `index_profile.html`.
- Built-in search (`themes/PaperMod/layouts/search.html` + `index.json`) indexes title/content/summary only — it does **not** index tags, so the search box won't match on tag names.

## Working with this repo

- Dev server: `hugo server` (or `hugo server -D` to include drafts) from the repo root.
- Production build: `hugo` (outputs to `public/`); `hugo --minify` for a minified build.
- Test a template change quickly without touching `public/`: `hugo --minify --destination <tmp-dir>` and grep the generated HTML.
- Theme lives in a submodule — if `themes/PaperMod` ever looks empty/stale, it needs `git submodule update --init`.
