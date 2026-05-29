# 🤝 Contributing to voxihost-blog-content

We are excited that you want to help improve the VoxiHost knowledge base and blog articles! Your contributions keep our guides up-to-date, accurate, and helpful for the entire hosting and development community.

By submitting corrections or new articles, you help other developers, system administrators, and tech enthusiasts. In return, we highlight your contributions on the official VoxiHost blog!

---

## 🛣️ Two Contribution Paths — Which One Is Yours?

There are two completely different workflows depending on what you want to do. Read both and pick the right one:

| I want to… | Path | Where to work |
| :--- | :--- | :--- |
| Fix a typo, broken command, or outdated info | **✏️ Edit an existing post** | `src/published/<locale>/<slug>/` |
| Write a brand new guide or article | **📄 Write a new article** | `src/drafts/<locale>/<slug>/` |

> [!IMPORTANT]
> **Never** submit new articles directly to `src/published/`. That directory is managed by the VoxiHost team. New articles always go to `src/drafts/` first.

---

## ✏️ Path A: Editing an Existing Article

Use this when you want to fix a typo, update a deprecated command, add a missing step, or correct outdated information in a post that is **already live on the blog**.

### Step-by-step

1. **Find the article** you want to edit in the published posts table in [`README.md`](README.md), or browse directly to:
   ```
   src/published/
   ├── en/           ← English posts
   │   └── <slug>/
   │       └── <slug>.md   ← Edit this file
   └── pl/           ← Polish posts
   ```
2. **Open the `.md` file.** You can edit it:
   - **Directly on GitHub:** Click the pencil icon (✏️ **Edit this file**) in the top right corner of the file view. No local setup needed.
   - **Locally:** Clone the repo, edit with any text editor, commit and push.
3. **Make your changes.** Common types of edits:
   - Fixing typos or grammar
   - Updating package version numbers or deprecated flags
   - Adding a missed step or clarification
   - Updating a broken link
   - Adding or replacing a screenshot (put new images in `images/` and reference them with `{% image %}` shortcode — see [FORMATTING.md](FORMATTING.md))
4. **Do NOT touch the YAML frontmatter** unless you are adding your GitHub username to `contributors` or updating the `updated: YYYY-MM-DD` date field.
5. **Submit a Pull Request** with a short description of what you changed and why.

### Key rules for edits
- **Preserve existing formatting** — do not restructure headings or reorder steps unless specifically fixing a logic error.
- **One fix per PR** — keep your changes focused. A PR fixing 3 unrelated articles is harder to review and more likely to be rejected.
- **Add your username** to the `contributors` array in the YAML frontmatter if it's not already there.
- **Update the `updated:` field** in the YAML frontmatter to today's date if you made a substantial content change (not for pure typo fixes).

---

## 📄 Path B: Writing a New Article

Use this when you want to contribute an **original, complete guide** that does not yet exist on the VoxiHost blog.

### Step-by-step

1. **Choose your article slug.** The slug is the URL-friendly name of your article. Rules:
   - Lowercase letters, numbers, and hyphens only: `install-nodejs-ubuntu-24`
   - No spaces, underscores, capital letters, or special characters
   - Should clearly describe the article topic
   - EN and PL versions of the same article have **different slugs** (but share the same `translationKey`)

2. **Create the folder structure** under `src/drafts/`:
   ```
   src/drafts/
   └── en/                              ← or 'pl/' for Polish
       └── install-nodejs-ubuntu-24/    ← your slug
           ├── install-nodejs-ubuntu-24.md   ← MUST match folder name exactly
           └── images/                       ← all screenshots/diagrams go here
   ```
   > [!IMPORTANT]
   > The `.md` file name **must exactly match** the folder name. If your folder is `install-nodejs-ubuntu-24/`, the file inside must be `install-nodejs-ubuntu-24.md`. This is how the blog engine finds the article.

3. **Copy the article template.** Start from [ARTICLE_TEMPLATE.md](ARTICLE_TEMPLATE.md) in the root of the repository. It has all YAML fields pre-filled with inline comments. Do not start from scratch.

4. **Fill in the YAML frontmatter.** Every required field must be present. See [CONTRIBUTING.md → YAML Reference](#2-yaml-frontmatter-complete-reference) or [FORMATTING.md](FORMATTING.md) for details.

5. **Write your content.** Follow the styling rules in [FORMATTING.md](FORMATTING.md):
   - Use `{% image %}` shortcodes, never `![alt](url)` syntax
   - Wrap the brand name in HTML spans
   - Use `>` blockquotes for tips and warnings
   - Include enough images that the guide isn't a "wall of text"

6. **If writing in both languages** (recommended!), create two separate folders — one under `en/` and one under `pl/` — using different slugs but the **same `translationKey` value** in both files. This activates the 🇬🇧/🇵🇱 language switcher on the published page.

7. **Submit a Pull Request** targeting the `src/drafts/` path. Include in your PR description:
   - What the article covers
   - Which OS/versions you tested the commands on
   - Whether you plan to write the PL version too (or if you'd like help with translation)

### Key rules for new articles
- **All commands must be tested.** No copy-pasting from ChatGPT or other sources without personal verification. See [AI-Assisted Writing Policy](#4-ai-assisted-writing-policy).
- **One article per PR.** Do not submit multiple unrelated articles in a single PR.
- **Article must be complete.** Do not submit an outline or half-finished draft — the article must be ready to publish from start to finish.
- **Your username in `contributors`.** Add your GitHub username to the `contributors` array so your avatar appears on the article page.

---

## ⏱️ What Happens After You Submit a PR?

Regardless of which path you chose, the review process is the same:

| Step | Who | What happens |
| :--- | :--- | :--- |
| 1. PR submitted | You | GitHub automatically shows the PR checklist from `.github/PULL_REQUEST_TEMPLATE.md` |
| 2. Editorial review | VoxiHost Team | We review content accuracy, formatting, and brand standards (usually within a few days) |
| 3. Feedback or merge | VoxiHost Team | We either merge your PR, or leave review comments requesting changes |
| 4. Publishing (new articles only) | VoxiHost Team | We move the file from `drafts/` → `published/`, compile ZIP packages, and deploy |
| 5. Attribution | Automatic | Your GitHub avatar and profile link appear on the article page immediately after deploy |

---

## 🛠️ Technical and Formatting Guidelines

To ensure the articles compile and render correctly on the VoxiHost blog engine, please follow these rules. The blog is built with **Eleventy (11ty)**, processes articles from ZIP packages, and uses a custom Nunjucks image shortcode — so there are a few specific patterns you must follow.

### 1. Article Template

The fastest way to start is to **copy the ready-made article template** from the root of this repository:

👉 **[ARTICLE_TEMPLATE.md](ARTICLE_TEMPLATE.md)**

It contains a fully pre-filled frontmatter block with inline comments explaining every single field, and a structured content skeleton. Just copy, rename, and fill in your content.

### 2. YAML Frontmatter (Complete Reference)

Every `.md` file starts with a metadata block enclosed by triple dashes `---`. The table below lists every supported field:

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `title` | String | **Yes** | SEO-friendly article headline. Plain text only — no Markdown syntax. |
| `description` | String | **Yes** | Short summary (max 160 chars) for meta tags and article cards. Plain text only. |
| `image` | String | Team | Path to the hero/OG image. **Generated by the VoxiHost team** — leave as the placeholder or omit when submitting a draft. |
| `date` | Date | **Yes** | Publication date, format: `YYYY-MM-DD` |
| `updated` | Date | No | Last updated date, format: `YYYY-MM-DD`. Displays an "(Updated: …)" badge. |
| `translationKey` | String | **Yes** | Unique key shared between EN and PL versions of the same article (enables language switcher). |
| `locale` | String | **Yes** | Language code: `en` or `pl`. |
| `category` | String | **Yes** | Topic category. Must match one of the valid values below. |
| `tags` | Array | No | Lowercase kebab-case tags for search and filtering. |
| `status` | String | **Yes** | Use `draft` when submitting. The team changes it to `published` after review. |
| `author` | Object | **Yes** | Author object with `name` and `link`. See format below. |
| `contributors` | Array | **Yes** | Your GitHub username(s). The site fetches your avatar automatically. |
| `howto` | Object | No | HowTo structured data for step-by-step guides (enables Google rich results). See template. |
| `isIndex` | Boolean | No | Set to `true` only for hub/index pages — hides from the main post listing grid. |

**Author Format (always use the object form):**
```yaml
author:
  name: VoxiHost Team
  link: https://voxihost.pl/
```

**Contributors Format:**
```yaml
contributors:
  - your-github-username
```

**Valid `category` values:**

| EN value | PL value | Use for |
| :--- | :--- | :--- |
| `Tutorials` | `Poradniki` | Step-by-step technical how-to guides |
| `Updates` | `Nowości` | VoxiHost product announcements and devblogs |
| `Comparisons` | `Porównania` | Detailed side-by-side technology comparisons |

> [!NOTE]
> New categories may be added by the VoxiHost team in the future. If you believe your article doesn't fit any existing category, mention it in your PR description.

### 3. Styling & Formatting Guidelines

All articles must follow the VoxiHost Markdown styling and brand patterns.

> [!IMPORTANT]
> For the full reference on image shortcodes, brand logo HTML wrapping, auto-colored links, code blocks, and blockquotes, read:
>
> 👉 **[FORMATTING.md](FORMATTING.md)**

### 4. AI-Assisted Writing Policy

> [!NOTE]
> We are fine with AI tools (ChatGPT, Claude, Gemini, Copilot) helping you **draft, translate, or structure** your content. However, **unverified AI output is never acceptable**:
> - Every terminal command, script, configuration path, and package name **must be personally tested** in a real environment before submission.
> - PRs containing hallucinated, outdated, or non-functional commands will be **rejected immediately** — no exceptions.

### 5. Technical Accuracy & OS Compatibility
Your guide must be transparent about its scope:
*   **State what you tested on:** Explicitly mention the OS name, version, and architecture (e.g. `Ubuntu 24.04 LTS x86_64`, `Rocky Linux 9 aarch64`).
*   **Document differences:** If a command behaves differently on other systems (e.g., `apt` vs `dnf`, `firewalld` vs `UFW`, `systemd` vs `SysVinit`), document this using blockquote callouts or footnotes.

---

## 🏆 Contributor Recognition (Win-Win)

We want to make sure your contributions are recognized and credited fairly:

* **Minor Edits (Typos, Code Fixes, Command Updates):**
  At the bottom of the article on our website, we automatically generate a *"Contributors to this article"* section. This section displays links to the GitHub profiles of everyone whose PRs were merged into this post.
* **Major Rewrites & Substantial Updates:**
  If you significantly update or rewrite a large portion of an existing article (e.g., updating a guide for a new major OS release, rewriting sections to improve depth, or adding extensive troubleshooting sections):
  * You will be added as a **Co-Author** or added to the **contributors** array in the YAML frontmatter.
  * Your GitHub profile link and avatar will be dynamically pulled and displayed on the website.
  * The VoxiHost editorial team evaluates major rewrites case-by-case to ensure fair attribution.
* **Full Articles (Guest Writing):**
  If you would like to write and publish a complete, original guide with us:
  1. Add it as a draft in the `src/drafts/` directory via a Pull Request.
  2. Simply ensure your GitHub username is registered in the `contributors` array in the article's YAML frontmatter.
  3. Once approved, we will publish your article and feature your profile details on the official VoxiHost blog.
* **Long-Term Opportunities (Join the VoxiHost Team!):**
  We are always looking for passionate technical writers, Linux experts, and system administrators. If you become a frequent, high-quality contributor:
  * **Reviewer Roles:** We may reach out to invite you as a community reviewer, helping us validate and review other contributors' Pull Requests.
  * **Join the Team:** Exceptional long-term contributors will be prioritized and contacted for paid guest-writing projects or potential permanent roles in the VoxiHost team!

---

## 🛡️ Safety & Integrity Policy

To maintain the security, quality, and reputational integrity of the VoxiHost knowledge base, we enforce a strict contributor verification policy:

> [!WARNING]
> We do not accept contributions (Pull Requests) from accounts that:
> - Are suspicious, newly created, or show signs of bot/inauthentic activity.
> - Have zero or minimal credible public activity on GitHub.
> - Host, link to, or promote malicious, spam, or unethical repositories on their GitHub profile.
> - Are associated with plagiarism, copy-paste spam, or unverified AI-generated content dumps.
>
> All Pull Requests from accounts deemed untrustworthy or violating these safety guidelines will be rejected immediately without code review.

---

## ⚖️ Legal Agreement

By submitting a Pull Request (PR) to this repository, you agree to the following terms:

1. **Licensing:** You agree to license your contribution (text, code blocks, and graphics) under the **Creative Commons Attribution-NonCommercial 4.0 International (CC BY-NC 4.0)** license.
2. **Publication Consent:** You grant VoxiHost the right to publish, modify, and distribute your contribution on the official VoxiHost blog and associated channels.
3. **Copyright:** You represent that you are the author of the submitted content or possess full rights to distribute and license it. You agree not to submit any content that infringes upon third-party copyrights or intellectual property rights.
