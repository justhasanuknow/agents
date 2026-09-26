# Modern Static Professional Service Website: Build Instructions

This document provides detailed build instructions for an AI agent (Claude Code, Cursor, etc.) to produce a modern, SEO-compliant, dark/light mode capable static website for a professional service firm (law, consulting, engineering, accounting, architecture, healthcare, etc.).

---

## How to Use

1. Create an empty folder and start the agent there.
2. Provide this entire document to the agent along with the prompt below.
3. The agent will interview you, then build the site.

### Prompt to Give the Agent

> Build a modern, dark/light mode capable, SEO-compliant, fully responsive static website following this document.
>
> Workflow:
>
> **Phase 1 (Reference inspection)**: WebFetch the REFERENCE SITE (<https://karabogahukuk.com>) and study its architecture, file structure, CSS design system, JS modules, HTML patterns, and visual language. Also inspect inner pages (about, practice areas, team, contact). Do this silently. Do not describe findings to the user unless they ask.
>
> **Phase 2 (Interactive interview)**: Using the INTERVIEW PROTOCOL section as your script, ask the user the questions one at a time, in the order presented. For each question:
>
> - Show a clear, concise question
> - Include 1-2 examples of acceptable answers
> - Mark optional fields explicitly ("optional, press Enter to skip")
> - Validate responses (color contrast, URL format, email format) and ask again if invalid
> - For table-style inputs (services, team members), ask how many items the user wants, then ask each item's fields one at a time
> - Confirm critical inputs by repeating them back ("So your firm name is 'X', correct?")
>
> Do not proceed to build until all required information is collected. If user wants to skip an optional section, accept and move on. Save all collected answers to a local file `_client-info.md` (in the project root) so progress is preserved if interrupted.
>
> **Phase 3 (Build)**: Apply the TASK LIST sequentially. Strictly follow PROTECTED PATTERNS. Those are tested, working systems; do not deviate. Apply every item in SEO REQUIREMENTS. Commit logically at the end of each major step.
>
> **Phase 4 (Validation)**: Execute every item in the VALIDATION CHECKLIST. Report any items that cannot be auto-verified (visual things, third-party setup) and prompt the user to confirm them manually.
>
> **Phase 5 (Cleanup)**: Delete `_client-info.md` and this BUILD-TEMPLATE document from the project folder before final deploy.

---

## Reference Site

URL: <https://karabogahukuk.com>

This is a live example built with the same architecture. The agent must inspect:

- `/`: Homepage (hero + practice areas + about preview + contact info)
- `/pages/hakkimizda.html`: Vision/mission/values layout
- `/pages/faaliyet-alanlari.html`: Service card structure
- `/pages/ekibimiz.html`: Team cards
- `/pages/iletisim.html`: Form, FAQ, map
- `/en/` and its sub-pages: English mirror

Files to fetch and study:

- `css/style.css`: Design system + all component styles
- `js/theme.js`: Dark/light mode logic
- `js/contact-form.js`: Web3Forms integration
- `js/main.js`: Mobile menu, scroll effects

---

## Interview Protocol (Question Script)

This section is the agent's interview script. Ask the user each item in the order presented. Mark each as [REQUIRED] or [OPTIONAL]. For optional items, accept "skip" as an answer. For validations, re-ask if invalid.

Start the interview with the section "Language Preference" because it determines whether subsequent questions need primary + secondary inputs.

### Language Preference [REQUIRED, ask first]

- Ask: "Will the site be single-language or bilingual?"
- If bilingual, ask: "What is the primary language? And the secondary?"
- Default: single language (use the user's input language).
- All subsequent fields with "(secondary)" suffix are conditional on bilingual being selected.

### Firm

- **Firm name** (primary language) [REQUIRED]
- **Firm name** (secondary language) [REQUIRED if bilingual]
- **Industry** [REQUIRED]. Examples: law, consulting, engineering, accounting, architecture, healthcare. This determines schema.org type (LegalService, ProfessionalService, AccountingService, MedicalBusiness, etc.) and section naming ("Practice Areas" vs. "Services").
- **Tagline** (primary) [OPTIONAL]: short slogan for hero/footer
- **Tagline** (secondary) [OPTIONAL, if bilingual]
- **Founding year** [REQUIRED]: used in schema.org and copyright
- **Address** [REQUIRED]: street, district, city, postal code
- **Geo coordinates** [REQUIRED]: latitude, longitude (from Google Maps; right-click on location → coordinates). Validate format: two numbers between -90/90 and -180/180.
- **Domain** [REQUIRED]: used in canonical URLs and schema.org. Validate format: bare domain like `example.com` (no protocol).
- **Logo file path** [OPTIONAL]: if "none", a text-based logo using the firm name is generated
- **Hero background image** [OPTIONAL]: if "none", agent picks a tasteful generic image (or solid gradient)

### Services / Practice Areas [REQUIRED]

- First ask: "How many services do you want to list? (1-8, recommended 6-8). The 4 most important appear on the homepage; all appear in detail on the inner page."
- Then for each service, ask these fields one at a time:
  - **Font Awesome icon** [REQUIRED]: e.g., `fa-balance-scale`, `fa-briefcase`. Suggest options based on industry.
  - **Service name** (primary) [REQUIRED]
  - **Service name** (secondary) [REQUIRED if bilingual]
  - **Short description** (primary, ~1 sentence) [REQUIRED]: for homepage card
  - **Detailed paragraph** (primary, ~3-4 sentences) [REQUIRED]: for inner page
  - **Sub-services** (primary, 4-7 bullets) [REQUIRED]: for inner page list
  - **Anchor id** [OPTIONAL]: defaults to slugified service name (e.g., "criminal-law")

### Team Members [REQUIRED, at least 1]

- First ask: "How many team members? (1-5)"
- Then for each member, ask these fields one at a time:
  - **Full name** [REQUIRED]
  - **Title** (primary) [REQUIRED]: e.g., "Founding Partner"
  - **Title** (secondary) [REQUIRED if bilingual]
  - **Education** [OPTIONAL]: used in schema.org Person/alumniOf
  - **Specialties** [REQUIRED]: comma-separated, used in schema.org knowsAbout
  - **Bio** (primary, ~3 sentences) [REQUIRED]
  - **Bio** (secondary) [REQUIRED if bilingual]
  - **Photo file path** [OPTIONAL]: if omitted, a placeholder initial circle is used
  - **Instagram URL** [OPTIONAL]
  - **LinkedIn URL** [OPTIONAL]
  - **Email** [OPTIONAL]: if provided, agent must confirm: "Do you want this email visible on the site? (yes/no)"

### Contact [REQUIRED]

- **Phone numbers** [REQUIRED]: list of person + number pairs. Validate phone format (E.164 preferred: +90...).
- **Working hours** [REQUIRED]: e.g., "Monday-Friday: 09:00-18:00"
- **Social media accounts** [OPTIONAL]: platforms (Instagram, LinkedIn, X, etc.) with URLs. Ask one at a time per platform.
- **Google Maps embed URL** [REQUIRED]: full iframe `src` URL (instruct user: Google Maps → open location → "Share" → "Embed a map" → copy `src` from iframe code)

### Brand Colors [REQUIRED]

- **Primary** (light mode): hex code, main brand color
- **Secondary** (light mode): hex code, darker shade of primary (hover states)
- **Accent** (light mode) [OPTIONAL]: small accents
- **Primary** (dark mode): hex code. **Must have at least 4.5:1 contrast ratio with white text** (WCAG AA). Validate using a contrast formula; if invalid, warn the user and suggest a darker shade. Re-ask.
- **Secondary** (dark mode): hex code, darker shade of dark primary
- If user does not know, suggest defaults based on industry (e.g., navy + gold for law, teal + sage for healthcare).

### FAQ [OPTIONAL, recommended at least 3]

- First ask: "Do you want a FAQ section on the contact page? (yes/no)"
- If yes, ask: "How many questions?"
- Then for each Q&A, ask:
  - **Question** (primary)
  - **Answer** (primary, ~2-3 sentences)
  - **Question** (secondary) [REQUIRED if bilingual]
  - **Answer** (secondary) [REQUIRED if bilingual]

### Form / Spam Protection [REQUIRED]

- **Web3Forms Access Key** [REQUIRED]: obtain from [web3forms.com](https://web3forms.com) (email signup sufficient). Validate format: UUID-like string (8-4-4-4-12 hex characters).
- **hCaptcha** [OPTIONAL]. Ask: "Do you want hCaptcha bot protection on the contact form? (yes/no, default yes)". If yes, use Web3Forms default sitekey (`50b2fe65-b00b-4b9e-ad62-3ba471098be2`) unless user provides their own.
- **Email address registered with Web3Forms** [REQUIRED for testing]: agent uses this to inform user where form submissions will land. Does not appear on the site.

### Blog [OPTIONAL]

- Ask: "Do you want a blog/articles section? (yes/no, default no)"
- If yes:
  - **Ghost URL** [REQUIRED]: e.g., <https://blog.example.com>
  - **Content API Key** [REQUIRED]
  - **Category map** [REQUIRED]: mapping of services to Ghost tag slugs (agent constructs this from the Services list, asks user to confirm)
- If no: agent removes article pages and Ghost script references from the site.

### Footer Credit [OPTIONAL]

- Ask: "Do you want a 'Developed by [your name/agency]' credit in the footer? (yes/skip)"
- If yes: ask for credit text and URL.

---

## Architecture Overview

### Stack

- **Pure HTML + CSS + Vanilla JS**: no framework, no build step, no npm
- **Static hosting** compatible: GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc.
- **Form backend**: Web3Forms (no server, JS fetch only)
- **Bot protection**: hCaptcha (Web3Forms native support)
- **Blog backend**: (optional) Ghost CMS Headless API
- **Theme system**: CSS variables + `data-theme` attribute + FOUC-preventing inline script

### Folder Structure

```text
/
├── index.html                  # Primary language homepage
├── css/
│   └── style.css               # ~2000+ lines, design tokens + all components
├── js/
│   ├── main.js                 # Mobile menu, smooth scroll, header scroll effect
│   ├── theme.js                # Dark/light toggle (auto-injects button into navbar)
│   ├── contact-form.js         # Web3Forms integration
│   ├── ghost-config.js         # (optional) Ghost URL + API key + category map
│   └── ghost-content.js        # (optional) Ghost article rendering
├── pages/                      # Primary language inner pages
│   ├── about.html
│   ├── services.html           # Naming depends on industry: practice areas, services, etc.
│   ├── team.html
│   ├── articles.html           # (optional)
│   ├── article-detail.html     # (optional)
│   └── contact.html
├── en/                         # Secondary language mirror (if bilingual)
│   ├── index.html
│   └── pages/
│       ├── about.html
│       ├── services.html
│       ├── team.html
│       ├── articles.html
│       ├── article-detail.html
│       └── contact.html
├── img/                        # logo, team photos, office photos, hero bg, article placeholders
├── sitemap.xml                 # SEO
└── robots.txt                  # SEO
```

Use language-specific filenames as appropriate (e.g., `hakkimizda.html` for Turkish, `about.html` for English).

---

## Task List (Apply Sequentially)

### Step 1: Inspect the reference site

- WebFetch <https://karabogahukuk.com> homepage and inner pages
- Fetch the CSS and JS files (style.css, theme.js, contact-form.js, main.js)
- Document all patterns, design tokens, and naming conventions

### Step 2: Set up folder structure

- Create empty folders and files matching the structure above

### Step 3: CSS design system (`css/style.css`)

- Copy the design tokens from the reference site, **adapt color values to client brand colors**
- `:root` (light mode) + `[data-theme="dark"]` (dark mode) tokens
- Base styles (body, h1-h6, p, a, img, btn, container, section)
- Component styles (header, footer, cards, forms, etc.)
- Responsive media queries (992, 768, 576px breakpoints)
- `prefers-reduced-motion` support

### Step 4: Create JS files

- `js/theme.js`: copy from reference, only `STORAGE_KEY` may change
- `js/main.js`: copy from reference (mobile menu, smooth scroll, header scroll)
- `js/contact-form.js`: copy from reference, update `ACCESS_KEY` and `fromName` values

### Step 5: HTML base templates

- Common `<head>` template for all HTML files (meta + Open Graph + Twitter + canonical + hreflang + schema.org + FOUC script)
- Common `<header>` (logo + nav + language switcher + theme toggle insertion point)
- Common `<footer>` (4 columns + social media + copyright + credit)

### Step 6: Homepage (`index.html`, `en/index.html` if bilingual)

- Hero (large typography + CTA button)
- Practice areas grid (4 main services, each a clickable card)
- About preview (asymmetric 2-column: image + content)
- Contact info cards (address, phone, working hours)

### Step 7: Inner pages

- **About**: page-header + long descriptive paragraphs + vision/mission side-by-side + values list (with check-mark icons)
- **Services**: page-header + intro + 8 service detail cards (icon + h3 + paragraph + h4 "Services" + ul) + CTA section
- **Team**: page-header + intro + team cards grid
- **Contact**: page-header + intro + contact info cards (flex-wrap) + form + map iframe + FAQ (accordion)

### Step 8: Secondary language version (if bilingual)

- Translate all primary language pages
- `<html lang="en">` (or whichever secondary language)
- All text translated, aria-labels translated
- hreflang links bidirectional (each page links to all other language versions)

### Step 9: SEO files

- `sitemap.xml`: all page URLs with `<lastmod>` and `<priority>`
- `robots.txt`: sitemap reference + crawl permissions
- Favicon (16×16, 32×32, 180×180 Apple touch icon)

### Step 10: Test and validate

- Execute every item in the "VALIDATION CHECKLIST" section

---

## SEO Requirements (Mandatory)

The site must be SEO-complete. Every item below must be applied.

### Meta Tags (per page)

- `<meta charset="UTF-8">`
- `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
- `<meta name="description" content="...">`: page-specific, max 155 characters
- `<meta name="keywords" content="...">`: 5-10 page-relevant keywords
- `<meta name="author" content="...">`
- `<meta name="robots" content="index, follow">`
- `<title>`: page-specific, unique, max 60 characters

### Canonical and Multi-Language

- `<link rel="canonical" href="https://domain.com/full-path">`: every page
- `<link rel="alternate" hreflang="tr" href="...">`: for each language version
- `<link rel="alternate" hreflang="x-default" href="...">`: default version

### Open Graph (Facebook, LinkedIn, etc.)

- `<meta property="og:type" content="website">` (use `article` for article pages)
- `<meta property="og:url" content="...">`
- `<meta property="og:site_name" content="...">`
- `<meta property="og:title" content="...">`
- `<meta property="og:description" content="...">`
- `<meta property="og:image" content="...">`: recommended **1200×630px**
- For non-primary language pages, also include: `og:locale="en_US"` + `og:locale:alternate="..."`

### Twitter Card

- `<meta property="twitter:card" content="summary_large_image">`
- `<meta property="twitter:url" content="...">`
- `<meta property="twitter:title" content="...">`
- `<meta property="twitter:description" content="...">`
- `<meta property="twitter:image" content="...">`

### Schema.org JSON-LD (per page type)

- **Homepage**: `Organization` + `LegalService`/`ProfessionalService` (per industry)
  - name, url, logo, image, foundingDate, telephone, address (PostalAddress), geo (GeoCoordinates), openingHoursSpecification, sameAs (social media), areaServed, serviceType
- **Inner pages**: `BreadcrumbList` (Home → current page)
- **Team**: `Person` block per individual (name, jobTitle, worksFor, alumniOf, sameAs, knowsAbout)
- **About**: `Organization` + `BreadcrumbList`
- **Article detail**: `BlogPosting` (headline, description, image, datePublished, author (Person), publisher (Organization), articleSection, keywords)
- **Contact**: `LegalService`/`ProfessionalService` (full contact info) + `BreadcrumbList`
- Multiple JSON-LD blocks may be combined using `@graph` array

### Semantic HTML

- Use `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>` semantic elements
- **Exactly one `<h1>` per page** (inside page-header)
- Maintain heading hierarchy: h1 → h2 → h3 (do not skip levels)
- Wrap main content in `<main>` (a11y best practice)
- Use `<article>` for article-type pages

### Image SEO and Accessibility

- Every `<img>` must have a descriptive `alt` attribute (e.g., "Office interior of [Firm Name]")
- Use `loading="lazy"` on images below the fold (NOT on hero image)
- Include `width` and `height` attributes on `<img>` (prevents CLS: Cumulative Layout Shift, a Core Web Vital)
- Optimize image files (prefer WebP/AVIF; JPEG max 200KB for homepage)

### Site Files

- **`sitemap.xml`**: all pages listed with `<lastmod>` and `<priority>`

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
    <url>
      <loc>https://domain.com/</loc>
      <lastmod>2026-01-15</lastmod>
      <priority>1.0</priority>
    </url>
  </urlset>
  ```

- **`robots.txt`**, minimal:

  ```text
  User-agent: *
  Allow: /
  Sitemap: https://domain.com/sitemap.xml
  ```

### Accessibility (also impacts SEO)

- `aria-label` on icon-only buttons (mobile menu, theme toggle, social media)
- `aria-current="page"` on active navigation link
- `aria-live="polite"` on form status messages
- Color contrast minimum 4.5:1 (normal text), 3:1 (large text 18px+)
- Visible focus state: `:focus-visible` ring on all interactive elements

### Performance (Core Web Vitals)

- Preconnect to font origins:

  ```html
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  ```

- Font Awesome via CDN
- Google Fonts with `display=swap` parameter (prevents FOIT)
- JS files loaded with `defer`
- Third-party scripts with `async defer` (hCaptcha, etc.)

### Lighthouse Targets (production)

- Performance: ≥90 (mobile)
- Accessibility: ≥95
- Best Practices: ≥95
- SEO: 100

---

## Protected Patterns (Do Not Modify)

These patterns are tested in production. Do not deviate. Only adapt content.

### 1. Form Pattern (Critical: Windows Defender Phishing Prevention)

The HTML form **must not contain**:

- `<form action="https://...">` external POST URL
- `<input type="hidden" name="access_key" value="...">`
- `<input style="display:none" ...>` honeypot

These three patterns together trigger Defender's "Trojan:HTML/Phish" signature and cause file deletion.

**Correct pattern**:

```html
<!-- Clean, standard HTML form -->
<form id="contact-form">
  <input type="text" name="name" required />
  <input type="email" name="email" required />
  <!-- additional fields -->
  <div class="form-captcha">
    <div
      class="h-captcha"
      data-sitekey="50b2fe65-b00b-4b9e-ad62-3ba471098be2"
    ></div>
  </div>
  <div class="form-status" role="status" aria-live="polite"></div>
  <button type="submit">Send</button>
</form>
<script src="https://js.hcaptcha.com/1/api.js" async defer></script>
```

Inside `js/contact-form.js`:

```js
var ACCESS_KEY = "...";
formData.append("access_key", ACCESS_KEY);
formData.append("from_name", "...");
formData.append("botcheck", ""); // honeypot
fetch("https://api.web3forms.com/submit", { method: "POST", body: formData });
```

### 2. Dark/Light Theme with FOUC Prevention

Each HTML file must include this inline script before `</head>`:

```html
<script>
  (function () {
    try {
      var t =
        localStorage.getItem("STORAGE_KEY") ||
        (window.matchMedia &&
        window.matchMedia("(prefers-color-scheme: dark)").matches
          ? "dark"
          : "light");
      document.documentElement.setAttribute("data-theme", t);
    } catch (e) {}
  })();
</script>
```

Replace `STORAGE_KEY` with a firm-specific slug (e.g., `acme-theme`).

`js/theme.js` auto-injects the toggle button into the navbar and manages localStorage + system preference. See the reference implementation.

### 3. Glass Navbar

```css
header {
  position: sticky;
  top: 0;
  background-color: color-mix(in srgb, var(--bg-page) 85%, transparent);
  backdrop-filter: saturate(150%) blur(10px);
  -webkit-backdrop-filter: saturate(150%) blur(10px);
}
header.scrolled {
  background-color: color-mix(in srgb, var(--bg-page) 92%, transparent);
  border-bottom-color: var(--border-color);
  box-shadow: var(--shadow-sm);
}
```

### 4. Card Pattern (Consistent Across Components)

```css
.card {
  background-color: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  padding: var(--space-6) to var(--space-8);
}
.card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-md);
  border-color: var(--border-strong);
}
```

### 5. Grid Cells Must Have `min-width: 0`

In multi-column CSS Grid, if a grid cell contains fixed-width content (e.g., hCaptcha at 300px), the cell's intrinsic content size can push it beyond the viewport. Always add:

```css
.grid-child {
  min-width: 0; /* allow shrinking below content min-content */
}
```

### 6. Responsive Breakpoints

- **≥992px**: Desktop (2-column grids)
- **≤992px**: Tablet (grids become 1-column, mobile menu activates)
- **≤768px**: Mobile (hero shrinks, padding reduces)
- **≤576px**: Small mobile (container 92% width, single column)

Avoid adding new breakpoints. Use `flex-wrap` and `auto-fit`/`auto-fill` for organic responsive behavior.

### 7. Design Tokens (No Hardcoded Values)

Always use `var(--*)` for colors, spacing, radius, shadows. Do not use hardcoded `#ffffff`, `20px`, `8px`. Exceptions:

- Social media brand colors (`#3b5998` Facebook, etc.)
- Error state colors (`#d9534f`)
- Rare edge cases requiring specific values

---

## Code Conventions

### Naming

- CSS classes: kebab-case (`contact-card`, `practice-area-item`)
- CSS variables: semantic kebab-case (`--primary-color`, `--bg-elevated`, `--space-6`)
- JS variables: camelCase
- HTML ids: kebab-case, meaningful (`contact-form`, `practice-areas`)
- Filenames: kebab-case (`article-detail.html`)

### HTML Structure (every page)

```html
<!DOCTYPE html>
<html lang="...">
  <head>
    <!-- charset, viewport -->
    <!-- description, keywords, author, robots -->
    <!-- canonical, hreflang -->
    <!-- Open Graph -->
    <!-- Twitter Card -->
    <title>Page - Firm</title>
    <!-- CSS link -->
    <!-- Fonts preconnect + Google Fonts -->
    <!-- Font Awesome CDN -->
    <!-- Schema.org JSON-LD -->
    <!-- FOUC inline script (last, immediately before </head>) -->
  </head>
  <body>
    <header>
      <div class="container">
        <div class="logo">...</div>
        <nav>
          <button class="mobile-menu-btn">...</button>
          <ul class="menu">
            <li>...</li>
            <li class="lang-switcher">...</li>
          </ul>
        </nav>
      </div>
    </header>
    <section class="page-header">
      <!-- inner pages only -->
      <div class="container">
        <h1>Page Title</h1>
        <div class="breadcrumb">...</div>
      </div>
    </section>
    <!-- main content sections -->
    <footer>...</footer>
    <!-- scripts: main.js, theme.js, contact-form.js (if applicable), ghost-*.js (if applicable) -->
  </body>
</html>
```

### CSS Structure

```css
/* ===== Design Tokens ===== */
:root { ... }
[data-theme="dark"] { ... }

/* ===== Base ===== */
* { box-sizing }
body, h1-h6, p, a, img

/* ===== Layout ===== */
.container, .btn, section

/* ===== Header ===== */

/* ===== Hero / sections / cards ===== */

/* ===== Forms ===== */

/* ===== Footer ===== */

/* ===== Responsive ===== */
@media (max-width: 992px) { ... }
@media (max-width: 768px) { ... }
@media (max-width: 576px) { ... }
```

---

## Validation Checklist

### Visual

- [ ] Light/dark mode toggle visible and functional in navbar
- [ ] No theme flicker on page reload (FOUC)
- [ ] Mobile hamburger menu opens and closes correctly
- [ ] Hero is responsive, images don't shift unexpectedly
- [ ] Cards do not overflow at 320px-1920px viewport widths
- [ ] Dark mode primary color has at least 4.5:1 contrast with white text
- [ ] Consistent header and footer across all pages

### Functional

- [ ] Contact form submission shows green success message; email arrives via Web3Forms
- [ ] Submission rejected without completing hCaptcha
- [ ] Captcha resets after successful submission (can submit again)
- [ ] Footer credit link opens in new tab
- [ ] Language switcher redirects to same page in other language (hreflang correct)
- [ ] FAQ accordion opens and closes correctly
- [ ] All internal links resolve to correct pages
- [ ] Phone links use `tel:+...` format

### SEO

- [ ] Every page has unique `<title>` and `<meta description>`
- [ ] Canonical URLs are correct
- [ ] hreflang links reference each language version correctly
- [ ] Schema.org JSON-LD passes [validator](https://validator.schema.org/) without errors
- [ ] Open Graph and Twitter Card meta tags are correct
- [ ] `sitemap.xml` and `robots.txt` are present
- [ ] Every image has a descriptive alt attribute
- [ ] Heading hierarchy is correct (h1 unique per page, no level skipping)
- [ ] [PageSpeed Insights](https://pagespeed.web.dev/) report reviewed

### Security

- [ ] No form `action` attribute in HTML
- [ ] No hidden `access_key` in HTML
- [ ] No `style="display:none"` honeypot in HTML
- [ ] All external links use `rel="noopener noreferrer"`
- [ ] Web3Forms dashboard has production domain whitelisted

### Performance

- [ ] Lighthouse Performance ≥90 (mobile)
- [ ] Lighthouse SEO = 100
- [ ] Lighthouse Accessibility ≥95
- [ ] Images use `loading="lazy"` (except above-fold)
- [ ] Image files are optimized (max 200KB for homepage; prefer WebP/AVIF)
- [ ] CLS score is good (image `width`/`height` attributes present)

---

## FAQ

**The client's industry is not law.**
The architecture is industry-agnostic. In schema.org, replace `LegalService` with the appropriate type:

- Consulting: `ProfessionalService`
- Engineering: `ProfessionalService` + specific
- Accounting: `AccountingService`
- Healthcare: `MedicalBusiness`
- Restaurant: `Restaurant`
- Architecture: `ProfessionalService`

Adapt page names accordingly: "Practice Areas" → "Services", "Team" → relevant equivalent, etc.

**Single language only.**
Do not create `/en/` (or other language) folder. Remove the `lang-switcher` `<li>` from all HTML files. Remove `hreflang` links (keep only `canonical`). Exclude foreign-language URLs from sitemap.

**Client does not want a blog/articles section.**
Delete:

- `pages/articles.html`, `pages/article-detail.html`, and equivalents in other languages
- `js/ghost-config.js`, `js/ghost-content.js`
- All "Articles" links from page menus
- "Articles" links from footer

**Client does not want hCaptcha.**
Remove `<div class="h-captcha">...</div>` and the hCaptcha API script tag from HTML. Honeypot alone provides basic protection. If spam becomes a problem, re-enable hCaptcha.

**Form is not working.**
Check:

1. `ACCESS_KEY` is correct (`js/contact-form.js`)
2. Domain is whitelisted in Web3Forms dashboard (localhost for testing + production domain)
3. Web3Forms verification email has been confirmed after first submission
4. DevTools Network tab shows `api.web3forms.com/submit` returning 200

**Windows Defender deletes the HTML file or flags it as phishing.**
The HTML must not contain these patterns:

- `action=` external POST URL on form
- Hidden `access_key` or `from_name` inputs
- `style="display:none"` checkbox honeypot

All these must be in `js/contact-form.js` using `FormData.append()`.

**Dark mode color is hard to read with white text.**
Check `--primary-color` in `[data-theme="dark"]`. Contrast with white text must be at least 4.5:1. Avoid highly saturated colors; medium-dark teal/navy is safer (e.g., `#168a90`).

**Content overflows the viewport on mobile.**
Add `min-width: 0` to CSS Grid cells. If fixed-width elements (e.g., hCaptcha) are present, they must be allowed to shrink below their content min-content size.

---

## References

- Reference site: <https://karabogahukuk.com>
- Web3Forms: <https://web3forms.com> (form backend)
- hCaptcha: <https://www.hcaptcha.com>
- Schema.org Validator: <https://validator.schema.org>
- PageSpeed Insights: <https://pagespeed.web.dev>
- WAVE Accessibility: <https://wave.webaim.org>
- Lighthouse: Built into Chrome DevTools
- WebAIM Contrast Checker: <https://webaim.org/resources/contrastchecker/>
