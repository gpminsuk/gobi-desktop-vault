---
name: create-gobi-homepage
description: Generate or modify a Gobi Vault homepage (home.html) using one of 4 style templates (neon-terminal, minimal-editorial, magazine, brutalist) with an interview that customizes hero links, accent color, and fonts. All features (INFOVIZ filter, pagination, visualization viewer) are always included. Use when asked to create a vault homepage, redesign home.html, build a Gobi vault landing page, or apply a different style to an existing vault page.
metadata:
  version: 1.0.0
  author: jyk
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep]
---

# Create Gobi Homepage

Generate or rewrite the `home.html` rendered by Gobi Desktop for a vault page. The skill ships four distinct style templates and an interactive interview so each homepage can match the user's intent without rebuilding from scratch every time.

## Templates

| Slug | Aesthetic | Background | Accent | Display font | Body font |
|------|-----------|-----------|--------|--------------|-----------|
| `neon-terminal` | Cyberpunk / terminal | `#000000` | `#ccff00` neon lime | Space Grotesk | Inter |
| `minimal-editorial` | Quiet, refined, blog-like | `#fafaf7` warm cream | `#c8553d` terracotta | Inter | Crimson Pro (serif) |
| `magazine` | Editorial / NYT-style | `#f5f1ea` paper | `#1a1a1a` + `#b8860b` gold rule | Playfair Display | Inter |
| `brutalist` | Raw, high-contrast | `#ffffff` | `#ff3300` red | Inter Black (uppercase) | Inter |

All four implement the same gobi.* surface (hero, Articles grid, visualization viewer, INFOVIZ filter, pagination, footer). Style is the variable; functionality is constant — all features are always included.

### Style preview

Full-page renders with 8 stub Articles (1440×1024 viewport @2x, Playwright). Open the JPG to see the hero, Articles grid, and footer for each style.

| | |
|---|---|
| ![[examples/neon-terminal.jpg\|320]] | ![[examples/minimal-editorial.jpg\|320]] |
| **neon-terminal** | **minimal-editorial** |
| ![[examples/magazine.jpg\|320]] | ![[examples/brutalist.jpg\|320]] |
| **magazine** | **brutalist** |

Regenerate after editing any template: `python examples/_render_screenshots.py` (requires `pip install playwright && playwright install chromium`).

## Input

- Optional `--style=<slug>` — skip Q1 interview question.
- Optional `--from-scratch` — generate without using a template (pure interview-driven).
- Optional `--output=<path>` — default `App/home.html`.
- Existing `App/home.html` — used as the live reference if modifying in place.

## Output

- A single self-contained HTML file at the output path. All CSS/JS inline. CDN dependencies allowed for fonts, marked only.
- Append a row to `.gobi/logs/skill_usage_logs.md`: `| YYYY-MM-DD | HH:MM | create-gobi-homepage | <style> |`.

## Interview

Run sequentially. Skip a question if the user has already answered it (CLI args, prior turn, or explicit "use defaults"). Batch Q1+Q2 then Q3+Q4 to keep momentum.

**Q1 — Style** (skip if `--style` provided):
> Which look should the homepage have?
> 1. **neon-terminal** — current style (dark, neon lime, terminal/cyberpunk)
> 2. **minimal-editorial** — light, serif-bodied, calm blog feel
> 3. **magazine** — editorial paper, drop caps, gold-rule
> 4. **brutalist** — white + red, thick black borders, raw type
> 5. **custom-blend** — start from one of the above and override colors/fonts

**Q2 — Hero links** (always ask unless regenerating with `--reuse-links`):
> What buttons should sit in the hero next to the title? Provide a list of `{label, url}` pairs. Defaults if you skip: existing entries from current home.html (or empty if new). Set to empty list to remove external links entirely.

**Q3 — Brand overrides** (optional, default to template values):
> Any tweaks to the chosen style? You can override:
> - Accent color (hex; replaces template's `--accent`)
> - Display font (Google Fonts family; replaces template's display font)
> - Profile fallback emoji (used if `vault.thumbnailPath` is empty; default `🧠`)
> - Page title prefix (replaces `vault.title` in `<title>`; useful for custom branding)

**Q4 — Output path** (skip if `--output` provided):
> Where should the file land? Default `App/home.html`. Suggest `App/home-<style>.html` if previewing alternates side-by-side.

After collecting answers, summarize in 4-6 bullets and proceed unless the user pushes back.

**Q5 — Deploy** (always ask after file is written):

Before asking, resolve the actual vault slug from `gobi.vault.slug` (runtime) or `gobi vault status`. No need to add slug to `PUBLISH.md` — it's already available at runtime. Never show the placeholder `{vault-slug}` — always substitute the real value.

> Homepage is ready. `https://gobispace.com/@<actual-slug>` 에 배포할까요?
> 배포하면:
> 1. `App/home.html`을 webdrive에 sync (`gobi vault sync --upload-only`)
> 2. `PUBLISH.md`에 `homepage: "[[App/home.html?nav=false]]"` 설정 (없는 경우)
> 3. vault profile publish (`gobi vault publish`)
>
> 배포가 완료되면 홈페이지가 라이브로 공개됩니다.

If the user declines, skip deployment and just report the local file path.

## Workflow

1. **ANALYZE REQUEST**
   - If modifying in place, read current `App/home.html` to pull existing hero links / accent / title.
   - Run Interview (skip if all args provided).
   - Confirm summary if any answer materially changes the result (style switch, brand overrides, etc).

2. **LOAD TEMPLATE**
   - Read `templates/{style}.html` (or merge with current home.html for in-place mods).
   - For `custom-blend`, pick the closest base template and queue Q4 overrides.

3. **APPLY HERO LINKS**
   - Replace contents of the `<!-- HERO_LINKS:START -->…<!-- HERO_LINKS:END -->` block with one `<a>` per `{label, url}` pair from Q2. Match the template's button class (e.g. `btn outline`).

4. **APPLY BRAND OVERRIDES**
   - Q3 accent → patch `--accent` and `--accent-dim` in `:root`.
   - Q3 display font → patch the Google Fonts `<link>` and the relevant `font-family` declarations (`h1`, `section h2`, `.update-card h3`).
   - Q3 emoji → patch `<div class="profile-pic" id="profilePic">🧠</div>` default.
   - Q3 title prefix → patch `<title>` and the `vault.title` fallback in `updateHeroSection()`.

5. **VERIFY**
   - `grep "listBrainUpdates\|brainUpdateId\|listPersonalPosts"` → must be zero hits (deprecated names must not appear).
   - Open the file in a browser if possible (or report "manual visual check needed").
   - Confirm responsive `@media (max-width: 768px)` rules survived edits.

6. **WRITE & LOG**
   - Write to output path (`App/home.html` by default).
   - Append usage log row.
   - Tell the user: file written, what changed, how to preview locally (Gobi Desktop renders `App/home.html`).

7. **DEPLOY (Q5)**
   - Resolve the actual vault slug from `gobi.vault.slug` (runtime) or `gobi vault status`. No need to read slug from `PUBLISH.md`. Never display the placeholder `{vault-slug}`.
   - Ask the user via AskUserQuestion whether to deploy, showing the real URL (e.g. `https://gobispace.com/@cdc2026`).
   - If yes:
     1. Ensure `.gobi/syncfiles` contains `/App/**` (create if missing).
     2. Run `gobi vault sync --upload-only` to push `App/home.html` to webdrive.
     3. Ensure `PUBLISH.md` has `homepage: "[[App/home.html?nav=false]]"` in frontmatter (add if missing).
     4. Run `gobi vault publish` to update vault profile.
     5. Report success with a **clickable markdown link**: `[https://gobispace.com/@<actual-slug>](https://gobispace.com/@<actual-slug>)` so the user can open it directly.
   - If no: skip, report local path only.
   - **Important**: `.gobi/` internal files are NOT synced by gobi vault sync. The homepage must live in a vault-root-level folder like `App/`.

## Latest gobi-cli API (post v1.3.x)

Use these names in any newly written or modified JS. Full reference: [[reference/gobi-api]].

| Method | Sync? | Returns | Notes |
|--------|-------|---------|-------|
| `gobi.vault` | sync | `{vaultId, title, description, thumbnailPath, tags, ownerName, ownerProfilePictureUrl, webdriveUrl, slug}` | New props: `tags`, `slug`, `ownerProfilePictureUrl`. |
| `gobi.readFile(path)` | async | `Promise<string>` | Throws on not found. |
| `gobi.listFiles(folderPath)` | async | `[{name, type}]` | Primary method for listing Articles. |
| `gobi.fileExists(path)` | async | `boolean` | |

### Data source: `Articles/` folder

The homepage lists articles from the vault's `Articles/` folder — **not** from GobiSpace posts. Use `gobi.listFiles("Articles")` to enumerate `.md` files, then `gobi.readFile(path)` to read each article's frontmatter (title, tags, created date) and preview content. This is the user's knowledge collection: articles they authored, interactive data visualizations, and presentations.

**URL params**: Footer "POWERED BY GOBI" uses `?og=1` to surface OG meta.

**CDN whitelist**:
- Fonts: `https://fonts.googleapis.com` / `https://fonts.gstatic.com`
- Markdown: `https://cdn.jsdelivr.net/npm/marked/marked.min.js`
- Editorial templates may also load: `https://fonts.googleapis.com/css2?family=Playfair+Display`, `family=Crimson+Pro`.

## Caveats

### Single-file rule
Every template stays a single HTML file with inline CSS and JS. Never split into external files. CDNs OK only for the whitelist above.

### Markdown rendering (shared across templates)
- Custom `marked.Renderer`:
  - GobiSpace file links (`gobispace.com/@slug?file=path`) → intercept, call `openFileViewer(path)` for inline viewer.
  - Relative-path links (no `http`) → also route through `openFileViewer`.
  - All other external links → `target="_blank" rel="noopener"`.
- Images:
  - Relative-path image (`_files_/img.png`) → resolved via `getFileUrl(path)` to webdrive URL.
- Article full view uses `marked.parse(processed)` after `resolveWikiLinks` + `resolveWikiImages`.
- Article preview uses `marked.parse(escapeHtml(content.substring(0,300)))`.

### Visualization viewer (`openFileViewer`)
- Normalize path: strip query params + decode `%20` / `+`.
- If extension is `.html`: render inside `<iframe>` with white background and zero padding (full-bleed effect for interactive data viz / presentations).
- Otherwise: `gobi.readFile(path)` → strip YAML frontmatter → `marked.parse`.

### Filter system
- `setInfoVizFilter()` filters articles to viz/HTML artifacts. `clearFilters()` resets.
- INFOVIZ test: article content includes `.html` OR a tag matches `/viz/i` or `/인포/`.

### Article card interactions
- Click toggles preview ↔ full (`toggleArticleDetail`); link clicks bail via `event.target.closest('a')` guard.
- Display max 3 topic badges per card (`tags.slice(0, 3)` from frontmatter).
- Full body uses a slightly brighter text (`#c0c0c0` in dark templates, `#3a3a3a` in light templates) for hierarchy.

### Layout (logical sections, in order)
```
Hero (single column)
├── profile-pic + h1 + hero-actions ({HERO_LINKS} + INFOVIZ)
├── hero-desc (vault.description)
About (rendered from PUBLISH.md body — markdown below frontmatter)
Overlays (z-index 1000): VISUALIZATION_VIEWER
Articles (articles-grid 2-col, 1-col on mobile)
Pagination ("More Articles")
Footer (POWERED BY GOBI → gobispace.com/@slug?og=1)
```

### About section
- Reads `PUBLISH.md` via `gobi.readFile('PUBLISH.md')`, strips YAML frontmatter, renders the remaining markdown body as the vault's about/introduction section.
- Placed between Hero and Articles. Styled as a single-column prose block matching the template's body font.
- If `PUBLISH.md` body is empty (only frontmatter), the About section is hidden.

### Mobile
- `@media (max-width: 768px)`: hero-actions full-width-stacked, updates-grid → 1 column.

### XSS / security
- `escapeHtml(text)` on every user-supplied or article-derived value rendered as text (titles, tags).
- Never inject raw article content without going through `marked.parse(resolveWikiLinks(resolveWikiImages(content)))`.

## Custom blend (Q1 option 5)

If the user picks `custom-blend`:
- Ask which base template to start from (Q1 again, options 1-4).
- Drive Q3 aggressively (accent + display font + body font + border thickness + hover style).
- Save the resulting file with a `-custom` suffix so it's clearly not one of the four canon templates.

## Related skills / docs

- `gobi:gobi-homepage` — gobi-cli's own homepage skill (harness user-invocable); canonical source of the `window.gobi.*` API. Consult it if this skill's API table looks drifted.


## When NOT to use

- For post drafting / publishing → use the gobi-cli `global create-post` workflow (see [[feedback_gobi_cli_publish_command]]).
- For the GobiSpace community web UI → not configurable from this skill; that's an upstream Gobi product surface.
- For generating post summary GIFs → use `create-gif-slides` skill.
