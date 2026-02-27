# Portfolio — Malladi Sairam Karthik

> Software Engineer · AI Systems & Backend Infrastructure

Single-page portfolio website built as a static HTML artifact. No frameworks, no build step, no JavaScript dependencies — one file, production-ready.

**Live:** [sairamkarthik.vercel.app](https://sairamkarthik.vercel.app) &nbsp;|&nbsp; **Source:** [github.com/Karthik0000007/portfolio](https://github.com/Karthik0000007/portfolio)

---

## Architecture

### High-Level Structure

```
┌─────────────────────────────────────────────────────────┐
│                      index.html                         │
│                                                         │
│  ┌───────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  <style>  │  │  <body>      │  │  <script>        │  │
│  │           │  │              │  │                   │  │
│  │  Tokens   │  │  Semantic    │  │  IntersectionObs  │  │
│  │  Reset    │  │  HTML5       │  │  Scroll-spy       │  │
│  │  Layout   │  │  Sections    │  │  Mobile nav       │  │
│  │  Cards    │  │  ARIA roles  │  │  Smooth scroll    │  │
│  │  Respond. │  │  Skip links  │  │  Reduced motion   │  │
│  │           │  │              │  │                   │  │
│  └───────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────┘
         │                │                  │
    Zero deps        Zero deps          Zero deps
    (inline)         (semantic)         (vanilla JS)
```

### Component Map

```
index.html
├── HEAD
│   ├── Meta tags (viewport, description, OG-ready)
│   ├── Google Fonts — Inter (preconnected)
│   ├── Font Awesome 6.4.0 (CDN)
│   └── <style> — ~860 lines inline CSS
│       ├── Design tokens (CSS custom properties)
│       ├── Reset & base styles
│       ├── Navigation (fixed, blur backdrop, scroll-spy)
│       ├── Hero (responsive, badge animation)
│       ├── Philosophy grid (3-column)
│       ├── Project cards (vertical stack)
│       ├── Timeline (experience, dot indicators)
│       ├── Skills grid (2×2)
│       ├── About (2-column)
│       ├── Contact cards
│       ├── Scroll animations (fade-up)
│       ├── Responsive breakpoints (900px, 768px, 480px)
│       └── prefers-reduced-motion support
│
├── BODY
│   ├── Skip-to-content link (accessibility)
│   ├── Navigation bar
│   │   ├── Brand mark ("MSK")
│   │   ├── Section links (6)
│   │   └── Mobile hamburger toggle
│   │
│   └── <main>
│       ├── Hero — name, tagline, role tags, CTAs, photo
│       ├── Engineering Philosophy — 3 principle cards
│       ├── Projects — 6 project cards with:
│       │   ├── Description + context
│       │   ├── Technical highlights (bullet list)
│       │   ├── Quantified results (metrics bar)
│       │   ├── Tech pills
│       │   └── GitHub links
│       ├── Experience & Education
│       │   ├── Timeline (3 entries, current indicator)
│       │   ├── Education card
│       │   └── Award card
│       ├── Skills — 4 system-layer cards + languages bar
│       ├── About — narrative + research interests
│       └── Contact — email, GitHub, LinkedIn cards
│
└── SCRIPT — ~60 lines vanilla JavaScript
    ├── Mobile nav toggle (hamburger ↔ X)
    ├── Close nav on link click
    ├── Smooth scroll (href="#section")
    ├── Active nav highlight (scroll position)
    └── IntersectionObserver fade-up animations
```

### Rendering Pipeline

```
Browser Request
      │
      ▼
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Vercel CDN │────▶│  index.html  │────▶│  First Paint │
│  (edge)     │     │  (single)    │     │  (~200ms)    │
└─────────────┘     └──────┬───────┘     └──────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        Google Fonts   Font Awesome   ME.jpg
        (preconnect)   (CDN, async)   (local)
              │            │            │
              └────────────┼────────────┘
                           ▼
                    ┌──────────────┐
                    │  Interactive │
                    │  (~400ms)    │
                    └──────────────┘
```

---

## Design Decisions & Trade-offs

### Single-file vs. Component Framework

| Factor | Single HTML file | React/Vue/Astro |
|--------|-----------------|-----------------|
| **Build step** | None | Required (Node, bundler) |
| **Deploy complexity** | Drop on any CDN | CI/CD pipeline needed |
| **Cold start** | ~200ms (1 request) | ~400ms+ (JS bundle parse) |
| **Maintainability** | Harder at >2000 LOC | Better for multi-page |
| **SEO** | Full SSR by default | Needs SSR/SSG config |
| **Dependencies** | 0 runtime, 2 CDN | 50+ node_modules |

**Decision:** Single file. This is a static portfolio — not a web app. The content is author-controlled, changes infrequently, and must load instantly. The trade-off is that CSS/HTML maintenance is manual, but with ~1400 lines total, that's acceptable.

### Inline CSS vs. External Stylesheet

**Chose inline** to eliminate one network request and guarantee atomic deployment — no FOUC (flash of unstyled content), no cache invalidation mismatch between HTML and CSS versions. Trade-off: can't cache CSS independently, but at ~25KB gzipped total, the entire page is smaller than most hero images.

### CDN Fonts vs. Self-hosted

**Chose Google Fonts CDN** with `preconnect` hints. Trade-off: introduces a third-party dependency (Google Fonts availability). Mitigation: the `font-family` stack includes `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` — if Google Fonts fails, the page renders with system fonts. No layout shift because `font-display: swap` is the default for Google Fonts.

### JavaScript: Vanilla vs. Library

Three behaviors required: mobile nav toggle, scroll-spy, fade-in animations. Total: ~60 lines. Loading React (45KB min-gzipped) or Alpine.js (15KB) for this would be engineering negligence. `IntersectionObserver` is supported by 97%+ of browsers. No polyfills needed.

### Dark Theme Only vs. Theme Toggle

**Chose dark-only.** Rationale:
- Target audience is technical recruiters viewing on modern screens
- Eliminates theme state management, localStorage, and flash-on-reload bugs
- One design to maintain, one design to QA
- Dark themes have lower power consumption on OLED screens (increasingly common)

Trade-off: users who prefer light mode can't switch. Acceptable for a portfolio that will be viewed for 2–5 minutes.

### Image: Local vs. CDN

Profile photo (`ME.jpg`) is served from the same origin. Trade-off: no automatic format negotiation (WebP/AVIF). But it's a single 220×220 image — the optimization ceiling is ~15KB savings, not worth adding a build step or image CDN dependency.

---

## Failure Scenarios Considered

| # | Failure | Impact | Mitigation |
|---|---------|--------|------------|
| 1 | **Google Fonts CDN down** | Inter font doesn't load | CSS font stack falls back to system fonts (`-apple-system`, `Segoe UI`, sans-serif). Layout remains stable — no shift. |
| 2 | **Font Awesome CDN down** | Icons don't render | Icons are decorative, not functional. Navigation text labels remain. Screen readers use `aria-label` attributes, unaffected. |
| 3 | **JavaScript disabled** | No animations, no mobile nav, no scroll-spy | All content is visible in static HTML. Navigation links work (native anchor scroll). Only mobile hamburger menu is inaccessible — content is still scrollable. |
| 4 | **ME.jpg fails to load** | Broken image in hero | `alt` text ("Portrait of Malladi Sairam Karthik") displays. Fixed `width`/`height` attributes prevent layout shift. Hero content (name, tagline, CTAs) remains fully functional. |
| 5 | **Vercel edge goes down** | Site unreachable | GitHub repo is public — content is always accessible via source. Static HTML can be re-deployed to any CDN (Netlify, Cloudflare Pages, S3) in <5 minutes with zero config changes. |
| 6 | **Browser doesn't support `backdrop-filter`** | Navbar loses blur effect | Falls back to solid `rgba(9,9,11,0.85)` background. Fully opaque — still readable. No functional impact. |
| 7 | **`IntersectionObserver` unsupported** | Fade-up animations don't trigger | Elements remain at `opacity: 0` (bad). Mitigation: `prefers-reduced-motion` media query sets `opacity: 1` and `transform: none`, and this query also fires on older browsers that don't support the Observer API. Worst case: content is in DOM and accessible via scroll. |
| 8 | **User has `prefers-reduced-motion: reduce`** | All animations disabled | Explicitly handled in CSS — fade-ups show immediately (`opacity: 1, transform: none`), pulse animation stops. Intentional, not a bug. |

---

## Setup & Development

### Prerequisites

- Any modern web browser (Chrome 80+, Firefox 78+, Safari 14+, Edge 80+)
- Git (for version control)
- A text editor (VS Code recommended)

### Local Development

```bash
# Clone the repository
git clone https://github.com/Karthik0000007/sairamkarthik.git
cd sairamkarthik

# Open in browser — no build step, no server required
start index.html          # Windows
open index.html           # macOS
xdg-open index.html       # Linux
```

For live-reload during development (optional):

```bash
# Using Python (built-in)
python -m http.server 3000
# → http://localhost:3000

# Using Node.js
npx serve .
# → http://localhost:3000
```

### Deployment

The site is deployed on **Vercel** via GitHub integration. Every push to `master` triggers an automatic deployment.

```bash
# Make changes
# ...edit index.html...

# Commit and push
git add -A
git commit -m "Update: description of change"
git push origin master
# → Vercel auto-deploys in ~15 seconds
```

**Manual deployment to any static host:**

```bash
# The entire site is one HTML file + one image
# Upload index.html and ME.jpg to any static file server

# Netlify (drag & drop)
# → Drop the folder at app.netlify.com/drop

# Cloudflare Pages
# → Connect GitHub repo, set build command to empty, output dir to /

# GitHub Pages
# → Settings → Pages → Deploy from branch → master → / (root)
```

### File Structure

```
portfolio/
├── index.html      # Complete portfolio (HTML + CSS + JS)
├── ME.jpg          # Profile photo (220×220, served locally)
├── README.md       # This file
└── .gitignore      # Excludes legacy/working files
```

---

## Performance

| Metric | Value | Notes |
|--------|-------|-------|
| **Total size** | ~45KB (uncompressed) | HTML + inline CSS + inline JS |
| **Gzipped** | ~12KB | What actually transfers over the wire |
| **Requests** | 4 | HTML, Google Fonts CSS, Font Awesome CSS, ME.jpg |
| **First Paint** | ~200ms | Single HTML file, inline styles |
| **Interactive** | ~400ms | After fonts + FA load |
| **Lighthouse** | 95+ (estimated) | Static HTML, semantic markup, ARIA labels |
| **Build time** | 0ms | No build step |

---

## Browser Support

| Browser | Minimum Version | Notes |
|---------|----------------|-------|
| Chrome | 80+ | Full support |
| Firefox | 78+ | Full support |
| Safari | 14+ | Full support |
| Edge | 80+ | Full support (Chromium-based) |
| Mobile Safari | iOS 14+ | Responsive layout, touch targets |
| Chrome Android | 80+ | Full support |

Graceful degradation for older browsers: content is always accessible, animations and blur effects degrade safely.

---

## Accessibility

- **Skip-to-content link** — keyboard users can bypass navigation
- **Semantic HTML5** — `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`
- **ARIA attributes** — `aria-label` on navigation, toggle button; `aria-expanded` on mobile menu
- **Color contrast** — text meets WCAG AA on dark background
- **`prefers-reduced-motion`** — all animations disabled when user prefers reduced motion
- **Alt text** — profile image has descriptive alt text
- **Focus indicators** — skip link visible on focus, native focus styles preserved

---

## License

This is a personal portfolio. The code structure is available for reference. Please don't copy the content (text, project descriptions, personal information) verbatim.
