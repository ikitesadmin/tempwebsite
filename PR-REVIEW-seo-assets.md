## Review: `fix/seo-assets` (a8878c8)

Adds robots.txt, sitemap.xml, 404.html, og-image.svg/.png to the static Netlify site. Assets are well-crafted and on-brand, but two blockers are **merge-order traps** — this PR delivers little value (and some harm) if it lands before its sibling PRs (`fix/stories-page`, `fix/index-page`).

### 🔴 Blockers

**1. Sitemap lists 10 `/stories/*` URLs that don't exist on `main`** — `sitemap.xml:14-62`
Those story pages live in `fix/stories-page`. Merging this first submits 10 dead URLs to Google (the opposite of the SEO goal, and it erodes crawl trust).
→ Merge `fix/stories-page` first, **or** drop the `/stories/*` `<url>` entries here and add them when those pages ship.

**2. `/success-stories` canonical mismatches the actual file + every link** — `sitemap.xml:7`
The file is `success-stories.html` and all in-site links use `.html`. The `netlify.toml` redirect (`/* → /:splat`, `force=false`) is an identity no-op, not a clean-URL rewrite — so there's no canonical tag to reconcile the two forms.
→ Change to `https://services.ikites.ai/success-stories.html` to match reality.

### 🟠 Should fix

**3. OG image is orphaned — no meta tags reference it.** There are zero `og:`/`twitter:`/`canonical` tags anywhere on `main`. The shipped PNG affects no social preview until `fix/index-page` adds the meta block. Either add `<meta property="og:image">` (+ width/height) to `index.html` / `success-stories.html` here, or sequence this PR after `fix/index-page`.

**4. `.skip-link` has no CSS rule** — `404.html:42`. `styles.css` defines no `.skip-link`, so it renders as a visible always-on link rather than the focus-reveal pattern. Add a visually-hidden `.skip-link` rule to `styles.css` (ideally site-wide) or remove it.

**5. `og-image.svg` is dead weight.** Social platforms only render the PNG; the SVG is referenced nowhere and its named fonts won't embed. Keep it as editable source, but don't deploy/reference it.

### 🟡 Minor

- 404 nav mixes clean-URL home links with the `.html` story link — align once URL form is settled.
- All `<lastmod>` hardcoded to `2026-06-20`; will silently go stale.
- `robots.txt` `Allow: /` is redundant (harmless).
- OG teal `#1fa7a6` drifts slightly from the site token `#1C8D90`.
- PNG is 208 KB — could be ~halved with pngquant/oxipng.

### ✅ Confirmed correct
OG dimensions **1200×630** · 404 `noindex` present · all 404 asset paths resolve · Netlify auto-serves root `404.html` (the `force=false` redirect doesn't interfere) · sitemap is well-formed XML · canonical domain consistent across files.

**Bottom line:** sequence this PR after `fix/stories-page` and `fix/index-page`, or trim the sitemap to existing pages and add the `og:image` meta tags here so the assets aren't dead on arrival.
