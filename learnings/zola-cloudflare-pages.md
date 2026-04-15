# Zola on Cloudflare Pages

## `get_url()` produces absolute URLs

Zola's `get_url()` generates absolute URLs using `base_url` from `zola.toml`. On Cloudflare Pages preview deployments, the actual domain is different (e.g., `abc123.pages.dev`), so all internal links (CSS, JS, menu, articles, tags, feeds) point to the production domain instead of the preview.

This causes two problems:
1. **CSS/JS blocked by CSP** — `style-src 'self'` rejects resources from a different origin
2. **Navigation broken** — clicking any menu link takes you to production instead of staying on the preview

## Fix: `zola build --base-url $CF_PAGES_URL`

Cloudflare Pages sets `CF_PAGES_URL` to the deployment's actual URL. Pass it to Zola:

```bash
if [ "$CF_PAGES_BRANCH" = "main" ]; then
  zola build
else
  zola build --base-url $CF_PAGES_URL
fi
```

This makes every `get_url()` call use the correct domain. On `main`, it falls back to `base_url` from `zola.toml`.

**Set this in the Cloudflare Pages build command** (dashboard or API), not in a separate build script.

## CSS/JS: root-relative paths as defense in depth

For CSS and JS specifically, root-relative paths (`/abridge.css` instead of `get_url(path="abridge.css")`) work on ANY domain without needing `--base-url`. Useful as a belt-and-suspenders approach for the most critical resources.

Override `templates/partials/head.html` and `templates/partials/head_js.html` in the blog repo to shadow the theme defaults.

## `zola serve` handles this automatically

`zola serve` overrides `base_url` with the local server URL (e.g., `http://127.0.0.1:8096`), so all `get_url()` calls work correctly during local development. No special flags needed.

## Don't override templates for navigation links

There are ~50 `get_url()` calls across the theme (base.html, head.html, head_js.html, social.html, seo.html, robots.txt). Overriding all of them with root-relative paths is maintenance hell. Use `--base-url` instead.

## TL;DR

| Situation | Solution |
|---|---|
| CF Pages preview | `zola build --base-url $CF_PAGES_URL` (in CF build command) |
| Production (`main`) | `zola build` (uses `base_url` from zola.toml) |
| Local dev | `zola serve` (auto-handles base_url) |
| CSS/JS specifically | Root-relative paths as defense in depth |
