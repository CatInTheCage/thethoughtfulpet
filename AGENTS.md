# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is **The Thoughtful Pet**, an Astro 7 static site for research-backed pet feeding guides and product reviews. It is built from the Astro blog starter template and deployed as a static site to GitHub Pages.

## Common commands

All commands run from the repository root:

```bash
npm install          # Install dependencies
npm run dev          # Start dev server at http://localhost:4321
npm run build        # Build production site to ./dist/
npm run preview      # Preview the production build locally
npm run astro ...    # Run Astro CLI commands (e.g., astro check)
```

Use `npm run build` to validate changes before committing. Use `npm run preview` to verify the built site, because the static output behavior differs from the dev server (especially around trailing slashes).

## Architecture

### Astro configuration

`astro.config.mjs` uses `output: 'static'` and `trailingSlash: 'always'`. This means every internal URL must end with `/` (e.g., `/blog/`, `/about/`, `/categories/best-picks/`). Paths without a trailing slash will 404 on the static build or on simple static hosts.

### Content collections

Blog posts live in `src/content/blog/` as Markdown or MDX files. The collection is defined in `src/content.config.ts` with this schema:

- `title`: string
- `description`: string
- `pubDate`: date
- `updatedDate`: date (optional)
- `heroImage`: string, defaults to `/images/hero.jpg`
- `categories`: array of category slugs, defaults to `[]`
- `author`: string, defaults to "The Thoughtful Pet Research Team"
- `readingTime`: number (optional)
- `featured`: boolean, defaults to `false`

Posts are queried with `getCollection('blog')` and sorted by `updatedDate || pubDate` descending. The post URL is always `/blog/{post.id}/` where `post.id` is the filename without extension.

### Pages and routing

- `src/pages/index.astro` — homepage with hero, category grid, latest articles, and research process section.
- `src/pages/blog/index.astro` — paginated-style list of all articles.
- `src/pages/blog/[...slug].astro` — dynamic article pages; passes the post and full post list to `BlogPost` layout.
- `src/pages/categories/[slug].astro` — dynamic category pages for `best-picks`, `pet-knowledge`, `problem-solving`, `care-guides`.
- `src/pages/about.astro`, `src/pages/contact.astro`, `src/pages/newsletter.astro`, `src/pages/privacy-policy.astro`, `src/pages/affiliate-disclosure.astro` — static pages.
- `src/pages/rss.xml.js` — RSS feed using `@astrojs/rss`.

### Layouts

- `src/layouts/Page.astro` — wrapper for static pages. Accepts `title`, `description`, `image`, `breadcrumbs`, and `showNewsletter` props. Renders a narrow container, optional breadcrumbs, page header, slot content, and a newsletter form.
- `src/layouts/BlogPost.astro` — wrapper for individual blog posts. Renders the post header, hero image, Markdown/MDX content, affiliate disclosure, newsletter form, and related articles.

### Components

- `src/components/Header.astro` — sticky header with paw-print SVG logo + text (`SITE_TITLE`) and mobile hamburger menu. Navigation links must keep trailing slashes.
- `src/components/Footer.astro` — site footer with logo, mascot SVG, explore/company/connect links, and affiliate disclosure.
- `src/components/BaseHead.astro` — shared `<head>` content including global CSS, meta tags, Open Graph, Twitter cards, and conditional GA4 snippet.
- `src/components/HeaderLink.astro` — active-state aware navigation link.
- `src/components/Breadcrumbs.astro` — schema.org breadcrumb list.
- `src/components/RelatedArticles.astro` — shows up to three related posts by category on article pages.
- `src/components/NewsletterForm.astro` — ConvertKit-ready email form; currently uses `PLACEHOLDER_CONVERTKIT_ACTION_URL`.
- `src/components/ProductCard.astro` — affiliate product card with placeholder Amazon tag.
- `src/components/FAQ.astro` — schema.org FAQ accordion.
- `src/components/AffiliateDisclosure.astro` — disclosure banner linking to `/affiliate-disclosure/`.
- `src/components/FormattedDate.astro` — date formatter.

### Styling

Global styles live in `src/styles/global.css`. The design system uses HSL CSS custom properties:

- `--background`: warm off-white
- `--foreground`: near-black
- `--primary`: terracotta / burnt orange (`17 58% 56%`)
- `--primary-dark`: darker terracotta for hover states
- `--secondary`: sage green
- `--secondary-dark`: darker sage
- `--muted-foreground`, `--border`, `--card`, etc.

Utility classes include `.container`, `.container-narrow`, `.section`, `.card`, `.badge`, `.btn`, `.btn-primary`, `.btn-secondary`, `.text-muted`, `.section-eyebrow`, `.affiliate-disclosure`, `.newsletter-form`, and `.faq-item`.

The site imports Google Fonts (`Inter` for body, `Playfair Display` for headings) and still includes unused Atkinson Hyperlegible font files in `src/assets/fonts/`.

### Constants and placeholders

`src/consts.ts` exports:

- `SITE_TITLE` = "The Thoughtful Pet"
- `SITE_DESCRIPTION`
- `SITE_URL`
- `AFFILIATE_DISCLOSURE`
- `GA_TRACKING_ID` = `'PLACEHOLDER_GA_TRACKING_ID'`

Before production launch, replace these placeholders:

- `GA_TRACKING_ID` in `src/consts.ts`
- `PLACEHOLDER_CONVERTKIT_ACTION_URL` in `src/components/NewsletterForm.astro`
- `PLACEHOLDER_AMAZON_TAG-20` in `src/components/ProductCard.astro`

### Images and assets

- `public/images/` — site images including hero, post hero images, about photo, and mascot SVG.
- `public/images/logo-horizontal.svg`, `public/images/logo-mark.svg`, `public/images/logo-horizontal.png`, `public/images/logo-mark.png` — these exist but are not used in the current header/footer.
- `public/favicon.svg` — bowl icon in brand color `#C86B4A`.
- `src/assets/` — leftover starter placeholder images and unused font files.

### Internal link convention

Because `trailingSlash: 'always'` is enabled, every hardcoded internal link must end with `/`:

- `/blog/`
- `/about/`
- `/categories/best-picks/`
- `/affiliate-disclosure/`
- `/privacy-policy/`
- `/newsletter/`

Dynamic post links use `/blog/${post.id}/` and category links use `/categories/${cat.slug}/`. The `HeaderLink.astro` active-state logic assumes trailing-slash URLs.

### Header logo

The header logo is an inline paw-print SVG icon plus the `SITE_TITLE` text. Earlier attempts used a detailed SVG logo file, but it was not legible at navigation sizes. Do not switch back to a raster or complex SVG logo without verifying readability at small sizes.

### Category system

Categories are hardcoded in three places and must stay in sync:

1. `src/pages/index.astro` — category grid
2. `src/pages/blog/index.astro` — filter pills
3. `src/pages/categories/[slug].astro` — category metadata and `getStaticPaths`

The valid category slugs are: `best-picks`, `pet-knowledge`, `problem-solving`, `care-guides`. Post frontmatter uses these exact slugs in the `categories` array.
