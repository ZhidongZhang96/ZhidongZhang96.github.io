# Astro Typefolio

## Framework

- **Astro 5** (SSG with islands architecture, Content Collections v3)
- **Tailwind CSS 4** (`@tailwindcss/vite` plugin + `@tailwindcss/typography`)
- **TypeScript 5.9** (strictest preset)
- **Node.js 22** / **pnpm** (see `.nvmrc` / `pnpm-lock.yaml`)
- Deployable to **Netlify**, **Vercel**, or any static host
- CI: `.github/workflows/ci.yml` (lint → type-check → build)

## Quick Commands

```bash
pnpm install          # Install dependencies
pnpm dev              # Dev server at localhost:4321
pnpm build            # Production build to ./dist/
pnpm blogbuild        # Generate Pagefind search index from ./dist/
pnpm check            # Astro type check + Biome lint
pnpm format           # Prettier format
```

## Project Structure

```
├── astro.config.ts          # All integrations, remark/rehype plugins, Vite config
├── src/
│   ├── site.config.ts       # ★ SITE CONFIG: author, URL, locale, menu, Giscus, etc.
│   ├── content.config.ts    # Content Collections schema (blog, tag, gallery)
│   ├── content/
│   │   ├── blog/            # Blog posts as .md or .mdx (filename = slug)
│   │   ├── tag/             # Tag metadata pages
│   │   └── gallery/         # Gallery collections (dir/index.md + images)
│   ├── pages/               # File-based routing
│   │   ├── index.astro          # Homepage: intro, pinned posts, timeline, manuscripts
│   │   ├── about.astro          # About page
│   │   ├── blog/[...page].astro # Blog index (paginated/yr groupings)
│   │   ├── blog/[...slug].astro # Single blog post
│   │   ├── projects/index.astro # Projects showcase
│   │   ├── projects/archive.astro
│   │   ├── gallery/index.astro  # Photo gallery
│   │   ├── tags/index.astro     # All tags with counts
│   │   ├── tags/[tag]/[...page].astro # Posts by tag
│   │   ├── rss.xml.ts           # RSS feed
│   │   ├── og-image/[...slug].png.ts # Dynamic OG images (Satori)
│   │   └── 404.astro
│   ├── layouts/
│   │   ├── Base.astro           # Root layout (head, header, footer, theme, pangu)
│   │   └── BlogPost.astro       # Blog post layout (masthead, TOC, TLDR, backlinks, giscus)
│   ├── components/
│   │   ├── BaseHead.astro       # <head> meta/OG/Twitter/canonical/RSS
│   │   ├── Search.astro         # Pagefind search modal (Ctrl+K)
│   │   ├── ThemeToggle.astro    # Dark/light toggle
│   │   ├── ThemeProvider.astro  # Inline theme init script (prevents FOUC)
│   │   ├── blog/
│   │   │   ├── Masthead.astro   # Post hero: cover img, title, date, reading time, tags
│   │   │   ├── PostPreview.astro # Compact listing: date + title + desc
│   │   │   ├── TOC.astro        # Table of Contents (nested, scroll-spy)
│   │   │   ├── TLDR.astro       # TL;DR summary block from frontmatter
│   │   │   ├── Backlinks.astro  # Cross-post backlinks
│   │   │   └── Giscus.astro     # GitHub Discussions comments
│   │   ├── home/
│   │   │   └── WhatsNewTimeline.astro # Vertical timeline
│   │   ├── projects/
│   │   │   ├── ProjectCard.astro
│   │   │   └── ArchivedProjectCard.astro
│   │   ├── gallery/
│   │   │   └── GalleryRail.astro # Horizontal scrollable + GLightbox
│   │   ├── layout/
│   │   │   ├── Header.astro     # Site nav + search + theme toggle
│   │   │   └── Footer.astro
│   │   ├── Paginator.astro      # Prev/Next pagination
│   │   ├── FormattedDate.astro
│   │   ├── InlineIconLink.astro
│   │   ├── SkipLink.astro
│   │   └── ...
│   ├── plugins/                 # Custom remark plugins
│   │   ├── remark-reading-time.ts
│   │   ├── remark-admonitions.ts     # :::note / :::tip / :::warning blocks
│   │   ├── remark-github-card.ts     # ::github{repo="..."} cards
│   │   └── remark-post-backlinks.ts  # Backlink recording
│   ├── styles/
│   │   ├── global.css           # ★ THEME: Tailwind v4 config, CSS vars (light+dark), prose, fonts
│   │   ├── blocks/search.css
│   │   └── components/
│   │       ├── admonition.css
│   │       ├── github-card.css
│   │       ├── projects-card.css
│   │       └── inline-icon-link.css
│   ├── utils/
│   │   ├── date.ts              # getFormattedDate(), collectionDateSort()
│   │   ├── backlinks.ts         # Post ID extraction and resolution
│   │   ├── domElement.ts        # DOM helpers (theme, class toggles)
│   │   ├── generateToc.ts       # Nested TOC from MarkdownHeading[]
│   │   └── remark.ts            # mdast helpers
│   ├── types.ts
│   ├── env.d.ts
│   └── data/
│       └── post.ts              # Data access: getAllPosts, getBacklinksForPost, etc.
├── public/
│   ├── assets/avatar.png
│   ├── fonts/                   # Local subset fonts (Noto Sans SC, Open Sans)
│   ├── icon.svg                 # Favicon source
│   └── social-card.png          # Default OG fallback
└── tailwind.config.ts
```

## Content Collections (src/content.config.ts)

### Blog Post Frontmatter
```yaml
---
title: "Post Title"                    # Required, max 60 chars
description: "SEO/preview summary"      # Required
publishDate: 2026-03-18                 # Required
updatedDate: 2026-03-19                 # Optional
tags: ["astro"]                         # Optional, lowercased+duduplicated
tldr: ["Point 1", "Point 2"]           # Optional TL;DR list
coverImage: { src: "./cover.png", alt: "..." }  # Optional
ogImage: "./custom-og.png"             # Optional (auto-generated if absent)
draft: false                            # Optional, default false
pinned: false                           # Optional, default false
giscus: true                            # Optional, default true
---
```

### Tag Frontmatter
```yaml
title: "Tag Display Name"       # Optional
description: "Tag intro text"   # Optional
```

### Gallery Frontmatter
```yaml
title: "Collection Title"
description: "Collection intro"
publishDate: 2026-04-06
tags: ["travel"]
images:
  - src: "./photo.jpg"
    title: "Photo Title"
    caption: "Optional caption"
    descPosition: "right"   # Optional: left | right | bottom
draft: false
```

## Key Features

- **Dark/Light theme**: `data-theme` attribute strategy, no FOUC
- **KaTeX math**: `$...$` / `$$...$$`, via remark-math + rehype-katex
- **Code blocks**: Expressive Code (github-dark/github-light themes, theme-synced)
- **Admonitions**: `:::note|tip|important|caution|warning` container directives
- **GitHub cards**: `::github{repo="user/repo"}` pulls live stats via API
- **Backlinks**: Auto-detected internal post links
- **TL;DR**: List of summary points before post body
- **Search**: Pagefind static search index (Ctrl+K / Cmd+K)
- **Comments**: Giscus (GitHub Discussions), theme-synced
- **OG images**: Auto-generated per-post via Satori
- **Image gallery**: Horizontal scroll rails with GLightbox
- **CJK/Latin spacing**: Automatic via Pangu.js
- **RSS, sitemap, robots.txt, PWA manifest**: Auto-generated
- **Pinned posts**: Showcased on homepage and blog index

## Configuration Points

| File | What to change |
|------|---------------|
| `src/site.config.ts` | Site URL, title, author, locale, menu links, Giscus |
| `astro.config.ts` | Integrations, remark/rehype plugins, manifest |
| `src/styles/global.css` | Theme colors, typography, prose, spacing |
| `src/pages/index.astro` | Homepage intro, What's New timeline items |
| `public/icon.svg` | Favicon / app icon source |
| `public/social-card.png` | Default OG image fallback |
| `public/fonts/` | Local font files and CSS |

## Build Pipeline

1. `astro build` → outputs static site to `./dist/`
2. `pagefind --site dist` → generates search index in `dist/pagefind/`
3. Deploy `./dist/` (GitHub Pages, Netlify, Vercel, etc.)
