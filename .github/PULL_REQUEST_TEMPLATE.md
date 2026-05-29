## 📋 Pull Request Checklist

Before submitting this PR, please verify all items below. Incomplete submissions may be closed without review.

---

### 📝 Article Content

- [ ] The article's content is **factually accurate** — I personally ran, tested, and verified every command, script, and configuration shown.
- [ ] The article is **free of hallucinated or unverified AI-generated content** — every command was tested in a real environment.
- [ ] The article clearly states **which operating systems and versions** it was tested on (e.g. Ubuntu 24.04 LTS, Rocky Linux 9).
- [ ] The writing is clear, concise, and avoids unnecessary "walls of text" — images/screenshots break up long sections.

---

### 🗂️ YAML Frontmatter

- [ ] `title` is present and descriptive.
- [ ] `description` is present, under 160 characters, and plain text (no markdown).
- [ ] `image` path is correct: `/assets/images/blog/<lang>/<slug>/og-image.png`.
- [ ] `date` is in `YYYY-MM-DD` format.
- [ ] `translationKey` is set and matches the paired translation (if applicable).
- [ ] `locale` is set to `en` or `pl`.
- [ ] `category` uses one of the valid values (`Guides` / `Poradniki`, `Updates` / `Nowości`, etc.).
- [ ] `author` is formatted as an **object** with `name` and `link`, not a plain string.
- [ ] `contributors` array contains my GitHub username.
- [ ] `status` is set to `draft` (VoxiHost team will change it to `published` after approval).

---

### 🖼️ Images

- [ ] **No native markdown image syntax** (`![alt](url)`) is used anywhere in the article body.
- [ ] All images use the `{% image "path", "alt text", "sizes" %}` Nunjucks shortcode.
- [ ] All image files are stored in the `images/` subdirectory of the article folder.
- [ ] Image paths in shortcodes match the pattern `/assets/images/blog/<lang>/<slug>/<filename>`.
- [ ] All images have meaningful, descriptive alt text (for SEO and accessibility).

---

### 💎 Brand & Formatting

- [ ] The brand name **VoxiHost** in article body text is wrapped in HTML spans:
  `<span class="text-white">Voxi</span><span class="text-amber-300">Host</span>`
  *(Exception: inside code blocks, systemd unit files, image alt text, or YAML fields — keep plain text there.)*
- [ ] Internal links use standard Markdown format and point to the correct paths:
  - Premium VPS → `/premium-vps/` or `/pl/premium-vps/`
  - Budget VPS → `/budget-vps/` or `/pl/budget-vps/`
  - Shield/DDoS → `/shield/` or `/pl/shield/`

---

### 📁 File Structure

- [ ] The article folder name is in **kebab-case** and matches the article's URL slug.
- [ ] The `.md` file is named **exactly** the same as the folder (e.g. `folder/folder.md`).
- [ ] This PR targets the `src/drafts/` directory (not `src/published/` — that is managed by the VoxiHost team).

---

### 📜 Legal

- [ ] I confirm that I am the original author of this content (or have full rights to submit it).
- [ ] I agree to license this contribution under the **CC BY-NC 4.0** license as described in `CONTRIBUTING.md`.
- [ ] This content does not infringe on any third-party copyright or intellectual property.
