# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```
npm install              # Install dependencies (Node >=22.12.0 required)
npm run dev              # Start dev server at localhost:4321
npm run build            # Production build to ./dist/
npm run preview          # Preview the built site locally
npm run astro -- check   # Type-check the project
```

## Architecture

This is an **Astro v6** static blog site (Bear Blog theme), deployed to Cloudflare Workers at `ai-invest.zju-shen2017.workers.dev`. Pure Astro — no React/Vue/Svelte framework. CSS is hand-rolled with custom properties, no Tailwind.

### Content flow

1. **Blog posts** live in `src/content/blog/` as `.md` or `.mdx` files.
2. Each post's frontmatter is validated by the Zod schema in `src/content.config.ts` (Astro v5+ `glob` loader pattern). Required fields: `title`, `description`, `pubDate`. Optional: `updatedDate`, `heroImage`.
3. The dynamic route `src/pages/blog/[...slug].astro` calls `getCollection('blog')` in `getStaticPaths()`, then passes each post to the `<BlogPost>` layout via `render(post)`.
4. The `<BlogPost>` layout (`src/layouts/BlogPost.astro`) handles hero image rendering, formatted dates, and the content `<slot />`.
5. `src/pages/blog/index.astro` renders the blog listing — `getCollection('blog')` sorted by `pubDate` descending.

### Page head / SEO

- **`src/components/BaseHead.astro`** is the single `<head>` block used by every page. It accepts `title`, `description`, and optional `image` props. It handles charset, viewport, favicon, canonical URL, Open Graph, Twitter Card, font preloading, GA4 injection, and global CSS import.
- **`src/consts.ts`** holds `SITE_TITLE` and `SITE_DESCRIPTION` — the single source of truth for SEO text. Pages import these and pass them to `<BaseHead>`. Changing them here updates every page.
- Site URL (`https://ai-invest.zju-shen2017.workers.dev`) is set in `astro.config.mjs` — canonical URLs, OG URLs, sitemap, and RSS feed all derive from this.

### Content types

Pages that are standard Astro pages (not content collection entries) must have their `<BaseHead>` called explicitly with title/description props. Content collection entries (blog posts) get their metadata from frontmatter, passed through the `<BlogPost>` layout, which in turn calls `<BaseHead>`.

## Key constraints

- **Node >=22.12.0** — enforced in `package.json` engines
- **No wrangler.toml in repo** — Cloudflare Workers deployment is configured externally
- **Content schema uses Astro v5+ glob loader** (`astro/loaders`), not the older `getCollection()` + `src/content/config.ts` pattern. The config file is `src/content.config.ts`.
- **Font files are local** (`.woff` in `src/assets/fonts/`), loaded via Astro's `fontProviders.local()` — don't add external font CDN links
