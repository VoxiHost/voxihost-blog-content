# 🔍 VoxiHost Blog — SEO Guide & Best Practices

This document explains exactly how the VoxiHost blog engine generates SEO metadata, structured data (JSON-LD), and social previews — and what **you as a content author** need to do (or avoid doing) to maximize search engine performance.

> [!IMPORTANT]
> Most SEO on the VoxiHost blog is **automatic**. The engine reads your YAML frontmatter and does the rest. Your job is to fill in the right values. This document tells you which ones matter most and why.

---

## 📋 Table of Contents

1. [How the SEO Engine Works](#1-how-the-seo-engine-works)
2. [Meta Tags — What Gets Generated](#2-meta-tags--what-gets-generated)
3. [JSON-LD Structured Data (Rich Results)](#3-json-ld-structured-data-rich-results)
4. [The HowTo Schema (Google Rich Steps)](#4-the-howto-schema-google-rich-steps)
5. [Open Graph & Social Previews](#5-open-graph--social-previews)
6. [Hreflang & Language Switching](#6-hreflang--language-switching)
7. [Writing for SEO — Content Best Practices](#7-writing-for-seo--content-best-practices)
8. [OG Image Requirements](#8-og-image-requirements)
9. [Common SEO Mistakes](#9-common-seo-mistakes)

---

## 1. How the SEO Engine Works

The blog is built with **Eleventy (11ty)**. Every time a new article ZIP is imported, the engine:

1. Reads your YAML frontmatter (`title`, `description`, `image`, `date`, etc.)
2. Generates all `<meta>` tags, Open Graph tags, and Twitter Card tags automatically
3. Generates 3 JSON-LD structured data blocks:
   - `BlogPosting` (or `TechArticle + BlogPosting` for Tutorials)
   - `HowTo` (only if you define a `howto:` block in frontmatter)
   - `BreadcrumbList` (always, automatically)
4. Handles canonical URL, hreflang tags, and `x-default` for bilingual articles

**You never write raw HTML SEO tags.** Everything flows from your Markdown frontmatter.

---

## 2. Meta Tags — What Gets Generated

The engine generates the following `<meta>` tags automatically from your frontmatter:

| Meta Tag | Source field | Notes |
| :--- | :--- | :--- |
| `<title>` | `title` | Appended with `" \| VoxiHost"` automatically |
| `<meta name="description">` | `description` | Plain text only, HTML stripped |
| `<meta name="keywords">` | `tags` array | Joined as comma-separated values |
| `<link rel="canonical">` | Page URL | Always auto-generated |
| `<meta name="robots">` | Automatic | `index, follow` for all published posts |
| `<meta name="author">` | Always `"VoxiHost"` | Fixed at organization level |
| `<meta http-equiv="content-language">` | `locale` | Set to `en` or `pl` |

### Title Best Practices

The `title` is the single most important SEO field. Rules:

- ✅ **Be descriptive and specific:** `How to Secure SSH on Ubuntu & Debian: The Complete Server Guide`
- ✅ **Include the main keyword near the start:** Don't bury it after "The Complete Guide to..."
- ✅ **Include the target OS/platform** when applicable: `...on Ubuntu 24.04`
- ✅ **Keep it under 60 characters** (Google truncates at ~60 chars in SERPs)
- ❌ **No markdown, HTML, or emoji** in the `title` field — it goes directly into `<title>` and JSON-LD

### Description Best Practices

The `description` is shown as the **snippet text** in Google search results.

- ✅ **Keep it between 120–160 characters** — shorter gets padded, longer gets truncated
- ✅ **Summarize what the reader will accomplish:** "Learn how to harden your SSH daemon..."
- ✅ **Include the primary keyword naturally**
- ❌ **No markdown formatting** — no `**bold**`, no backticks, no links
- ❌ **No duplicating the title verbatim**

---

## 3. JSON-LD Structured Data (Rich Results)

The engine automatically generates two baseline JSON-LD blocks on every article page:

### 3a. BlogPosting / TechArticle

```json
{
  "@type": ["TechArticle", "BlogPosting"],
  "headline": "Your title here",
  "description": "Your description here",
  "image": "https://voxihost.pl/assets/images/blog/<slug>/og-image.png",
  "author": { "@type": "Person", "name": "VoxiHost Team", "url": "https://voxihost.pl/" },
  "datePublished": "2026-05-28",
  "dateModified": "2026-05-28",
  "inLanguage": "en",
  "articleSection": "Tutorials",
  "wordCount": 1847,
  "keywords": "linux, vps, ubuntu"
}
```

> **Notice:** Articles with `category: Tutorials` or `category: Poradniki` receive the **dual type** `["TechArticle", "BlogPosting"]` which signals to Google that this is a technical how-to, potentially qualifying for enhanced rich results. `Updates` / `Nowości` posts get only `"BlogPosting"`.

**What you need to fill in to make this complete:**
- `title` → `headline`
- `description` → `description`
- `image` → `image` (must be correct path!)
- `date` → `datePublished`
- `updated` → `dateModified` (if omitted, defaults to `date`)
- `category` → `articleSection`
- `tags` → `keywords`
- `author` object → `author`

### 3b. BreadcrumbList

Generated automatically on every article. No frontmatter required:

```
VoxiHost → Blog → [Article Title]
```

This enables the **breadcrumb trail** displayed in Google search results under the page URL.

---

## 4. The HowTo Schema (Google Rich Steps)

For step-by-step tutorial guides, the engine supports the **HowTo** structured data schema. When properly configured, Google can display your article with an **expandable step-by-step panel** directly in search results, showing estimated completion time and individual steps.

**This is opt-in.** Add the `howto:` block to your frontmatter to activate it.

### Example frontmatter:

```yaml
howto:
  name: How to Secure SSH on Ubuntu & Debian
  totalTime: PT20M
  yield: "A hardened SSH daemon with key-based auth, restricted root login, and custom port"
  tool:
    - A VPS or dedicated server running Ubuntu 22.04+ or Debian 11+
    - An SSH client (terminal, PuTTY)
    - A user account with sudo privileges
  steps:
    - name: Step 1 — Update the System
      text: Ensure all packages are up to date before making configuration changes.
      url: step-1--update-the-system
    - name: Step 2 — Edit the SSH Configuration File
      text: Open /etc/ssh/sshd_config and modify the key security parameters.
      url: step-2--edit-the-ssh-configuration-file
```

### Field Reference

| Field | Format | Description |
| :--- | :--- | :--- |
| `name` | String | Full descriptive title of the how-to procedure |
| `totalTime` | ISO 8601 | Duration: `PT15M` = 15 min, `PT1H30M` = 1.5 hrs |
| `yield` | String | What the user ends up with after completing the guide |
| `tool` | Array of strings | Prerequisites / tools needed |
| `steps[].name` | String | Short step label (shown in Google's rich result panel) |
| `steps[].text` | String | Brief description of what this step does (1–2 sentences) |
| `steps[].url` | String | **Anchor ID** matching the heading in your Markdown article |

### Critical: Matching the `url` anchor to your headings

The `steps[].url` must match the **auto-generated anchor ID** that the markdown parser creates from your headings. The engine uses `markdown-it-anchor` which slugifies heading text:

| Markdown heading | Generated anchor ID |
| :--- | :--- |
| `## Step 1 — Update the System` | `step-1--update-the-system` |
| `## Step 2 — Edit SSH Config` | `step-2--edit-ssh-config` |
| `## Conclusion` | `conclusion` |

**Rule:** Replace spaces with hyphens, strip special characters except hyphens, make everything lowercase. An em-dash ` — ` becomes ` -- ` (two hyphens).

> [!NOTE]
> Only add `howto:` for genuine step-by-step guides. Do **not** add it to news posts, devblogs, or short announcements — Google may penalize misuse of the schema.

---

## 5. Open Graph & Social Previews

When your article is shared on social media (X/Twitter, Facebook, LinkedIn, Fluxer, Telegram), the engine generates a rich preview card using the following tags:

| OG Property | Source | Notes |
| :--- | :--- | :--- |
| `og:title` | `title` | |
| `og:description` | `description` | |
| `og:image` | `image` | Must be 1200×630px for best results |
| `og:image:width` | Fixed: `1200` | |
| `og:image:height` | Fixed: `630` | |
| `og:type` | `article` | Always set for blog posts |
| `og:locale` | `locale` | `en_US` or `pl_PL` |
| `article:published_time` | `date` | |
| `article:modified_time` | `updated` | Only if `updated:` is set |
| `article:author` | `author.link` or `author.name` | |
| `article:tag` | `tags` array | Each tag becomes a separate `article:tag` |

Twitter/X additionally uses `summary_large_image` card — this shows the full `image` as a large preview instead of a thumbnail.

---

## 6. Hreflang & Language Switching

If your article has both an EN and PL version with matching `translationKey`, the engine automatically generates:

```html
<link rel="alternate" hreflang="en" href="https://voxihost.pl/blog/secure-ssh-ubuntu-debian/" />
<link rel="alternate" hreflang="pl" href="https://voxihost.pl/pl/blog/jak-zabezpieczyc-ssh-ubuntu-debian/" />
<link rel="alternate" hreflang="x-default" href="https://voxihost.pl/blog/secure-ssh-ubuntu-debian/" />
```

This tells Google that these two pages are translations of the same content, preventing duplicate content penalties and enabling the correct language to be shown to users in different countries.

**What you must do:**
- Set the **same** `translationKey` value in both the EN and PL frontmatter
- Set the correct `locale: en` / `locale: pl` in each respective file

If you only write in one language, still include `translationKey` — it will just produce a single `hreflang` tag without a cross-language pair.

---

## 7. Writing for SEO — Content Best Practices

### Heading Structure

The `title` frontmatter field becomes the `<h1>` on the page. **Never use a `# H1` heading** inside the article body — it creates two H1s which is an SEO error.

```
title: Your Article Title    ← This becomes <h1>
                                Do NOT add another # H1 in the body

## Introduction              ← Start with H2
### Sub-section              ← Then H3
#### Nested point            ← H4 if needed
```

### Keyword Density and Natural Language

- Use your primary keyword in the **first paragraph** of the article
- Use it **2–4 more times** throughout the article, naturally
- Use synonyms and related terms (LSI keywords) — e.g. for "secure SSH": "SSH hardening", "SSH daemon", "sshd_config", "disable root login"
- Do **not** keyword-stuff — Google penalizes unnaturally repetitive text

### Article Length

| Type | Recommended length |
| :--- | :--- |
| Quick guides (e.g. "How to add a sudo user") | 600–1,000 words |
| Standard tutorials (e.g. firewall setup) | 1,200–2,000 words |
| Comprehensive guides (e.g. LAMP stack) | 2,000–4,000 words |
| Devblogs / announcements | 400–800 words |

> The engine automatically calculates reading time from word count and displays it in the article header (`readingTimeComputed`).

### Internal Linking

Always link to related VoxiHost articles and product pages within your content. Use standard Markdown links — they get **auto-colored** by the engine based on destination:

```markdown
# Link to a related article:
See our guide on [how to configure UFW](/blog/configure-ufw-ubuntu-debian/).

# Link to a product page (gets auto-colored amber):
Get started with a [Premium VPS plan](/premium-vps/).

# Budget VPS (auto-colored sky blue):
Or try our [Budget VPS options](/budget-vps/).

# Shield/DDoS protection (auto-colored emerald):
Combined with [VoxiHost Shield](/shield/) for DDoS protection.
```

### Image Alt Text

Every image must have descriptive alt text. The engine uses this for:
- Accessibility (screen readers)
- Google Image Search indexing

```jinja
{# Good — describes the specific screenshot content: #}
{% image "/assets/images/blog/your-slug/step2.png", "Terminal output showing SSH service successfully restarted with systemctl", "(max-width: 768px) 100vw, 800px" %}

{# Bad — too generic: #}
{% image "/assets/images/blog/your-slug/step2.png", "Screenshot", "(max-width: 768px) 100vw, 800px" %}
```

---

## 8. OG Image — Handled by the VoxiHost Team

The `image:` field in frontmatter controls the hero image, social media preview card, blog listing thumbnail, and the `BlogPosting` JSON-LD `image` property.

> [!NOTE]
> **You do not need to create the OG/hero image yourself.** The VoxiHost editorial team generates and adds the branded hero image during the review and publishing process.
>
> When submitting a draft, simply leave the `image:` placeholder as-is:
> ```yaml
> image: /assets/images/blog/your-article-slug/og-image.png
> ```
> We will replace it with the final branded image before the article goes live.

The images you *do* need to provide are **in-article screenshots and diagrams** (step-by-step screenshots, terminal output, configuration examples). These go in the `images/` subfolder of your article folder and are referenced using the `{% image %}` shortcode — see [FORMATTING.md](FORMATTING.md) for details.

---

## 9. Common SEO Mistakes

### ❌ Wrong `image` path format in frontmatter

The `image:` field must use the correct production web path. When submitting a draft, leave the placeholder — the VoxiHost team will set the correct path when publishing:

```yaml
# Wrong — local relative path:
image: images/og-image.png

# Wrong — missing slug folder:
image: /assets/images/blog/og-image.png

# Correct placeholder for your draft (leave as-is):
image: /assets/images/blog/your-article-slug/og-image.png
```

### ❌ `description` too short or too long

```yaml
# Too short (< 120 chars) — Google may auto-generate a worse snippet:
description: A guide to installing Docker.

# Too long (> 160 chars) — truncated in SERPs with "...":
description: This is an extremely detailed, comprehensive, step-by-step guide that covers every single aspect of Docker installation on Ubuntu and Debian-based Linux systems, including troubleshooting.

# Good (120-160 chars):
description: Learn how to install Docker Engine on Ubuntu 22.04 and Debian 12. Includes post-install steps, testing, and managing Docker as a non-root user.
```

### ❌ Using `# H1` heading in the article body

```markdown
# This Is My Article Title   ← NEVER DO THIS inside the body
                                The title: frontmatter is already H1

## Introduction              ← Always start here
```

### ❌ `howto.steps[].url` anchor doesn't match the real heading

```yaml
# Frontmatter says:
url: step-1-update-system

# But the heading in the article is:
## Step 1 — Update the System
# Which generates anchor: step-1--update-the-system  (note double dash!)

# Correct:
url: step-1--update-the-system
```

### ❌ Missing `updated:` field after substantial edits

When you significantly update an existing article, always add:
```yaml
updated: '2026-05-28'
```

This updates the `article:modified_time` OG tag and `dateModified` in JSON-LD, which tells Google the content is fresh and signals re-crawling.

### ❌ Writing only one language version

Having paired EN + PL articles with `translationKey` gives **double the indexed pages**, double the search real estate, and avoids duplicate content penalties via hreflang. If you write in English, consider also writing the Polish version (or asking the VoxiHost team for help with translation).
