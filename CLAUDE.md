# Zhidong Zhang Personal Blog

## Framework

- **Hugo** v0.128.0 Extended (no external theme — custom-built from scratch)
- Deployed to **GitHub Pages** via `.github/workflows/hugo.yaml`

## Project Structure

```
├── hugo.toml              # Core config (params, menus, markup, permalinks)
├── content/
│   ├── _index.md          # Homepage "About Me" body text
│   └── blogs/             # Blog posts as .md files
├── data/
│   ├── highlights.yml     # Timeline/news items
│   ├── publications.yml   # Academic publications
│   └── projects.yml       # Project portfolio
├── layouts/
│   ├── index.html         # Homepage template (renders data/*.yml sections)
│   ├── blogs/
│   │   ├── list.html      # Blog listing
│   │   └── single.html    # Blog post (TOC sidebar, sharing, code copy)
│   ├── tags/
│   │   ├── list.html      # Tag-filtered blog list
│   │   └── terms.html     # Tag cloud
│   ├── partials/
│   │   ├── head.html      # HTML <head> (FontAwesome, Academicons, KaTeX CDN)
│   │   └── sidebar.html   # Left sidebar (avatar, name, social, nav, CV)
│   └── shortcodes/
│       └── bookmark.html  # Bookmark card shortcode for inline link previews
├── static/
│   ├── css/style.css      # All site styles (~450 lines)
│   ├── images/            # Avatar, screenshots
│   ├── files/             # PDFs (thesis, slides)
│   ├── CNAME              # Custom domain: zhidongzhang.cn
│   └── cv.pdf             # Downloadable CV
└── .github/workflows/hugo.yaml
```

## Key Conventions

### Blog Posts
- Frontmatter format (YAML `---` delimiters):
  ```yaml
  ---
  title: "Post Title"
  date: 2024-01-25
  description: "Short summary"
  tags: ["Tag1", "Tag2"]
  slug: "optional-custom-url"
  ---
  ```
- Permalink pattern: `/blogs/:title/`
- **Math**: KaTeX loaded globally (all pages). Use `$$` for display math, `$` for inline math.
- **Highlighting**: Use `==text==` for highlighted text (rendered client-side to `<mark>`).
- **Code blocks**: Copy button added automatically client-side.

### Data Files (homepage sections)
- `data/highlights.yml` — list of `{date, description, url?}` entries
- `data/publications.yml` — list of `{title, authors, venue, year, links: [{name, url}]}`
  - Bold own name with `**Zhidong Zhang**` in authors
- `data/projects.yml` — list of `{title, year, note?, description, links, badges?}`

### Sidebar Configuration (`hugo.toml`)
- Social icons in `[[params.social]]` support `icon` from FontAwesome or Academicons (`pack = "ai"`)
- Icon packs loaded via CDN in `layouts/partials/head.html`:
  - FontAwesome 6.5.1 (`fas`, `fab`, `far` prefixes)
  - Academicons 1.9.4 (`ai` prefix)

### Markup
- Goldmark with `unsafe=true` (raw HTML allowed in markdown)
- Syntax highlighting: `style = "friendly"`, `guessSyntax = true`
- LaTeX passthrough: `$$`/`$$` and `\[`/`\]` for block; `$`/`$` and `\(`/`\)` for inline

## Build & Deploy

```bash
hugo server          # Local dev at http://localhost:1313/
hugo --gc --minify   # Production build to public/
```

Push to `main` auto-deploys via GitHub Actions. No `package.json` currently needed.
