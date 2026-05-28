# 📂 Source Content (`src/`)

This directory contains the raw Markdown source files and image assets for all VoxiHost blog articles and guides. 

## 🗂️ Directory Structure

* **`published/`**: Live blog posts that are published on the website.
  * Organized by locale: `en/` (English) and `pl/` (Polish).
  * Inside each locale, every post has its own folder named after its URL slug (e.g., `published/en/install-nginx-ubuntu-debian/`).
  * Each post folder contains the main `.md` file and a subfolder named `images/` for any graphics, screenshots, or diagrams.
* **`drafts/`**: In-progress drafts, guest article suggestions, and posts awaiting review or scheduled for release.
  * Organized similarly by locale (`en/`, `pl/`) and slug folder.
  * *Note:* Unlike published articles, drafts **do not** compile to ZIP files in the `dist/` directory.

## ✍️ How to Add or Edit an Article

1. **For Edits:** Locate the article's folder under `published/<locale>/<slug>/`, make your changes to the `.md` file, and commit.
2. **For New Articles:**
   * Create a folder under `drafts/<locale>/<slug-name>/`.
   * Create a Markdown file inside it named `<slug-name>.md`.
   * Fill out the required YAML frontmatter (with `status: draft`).
   * Put any screenshots/graphics inside a subfolder named `images/`.

For complete contribution requirements, formatting guidelines, and licensing terms, please refer to [CONTRIBUTING.md](../CONTRIBUTING.md) in the repository root.

> **New article?** Start by copying [ARTICLE_TEMPLATE.md](../ARTICLE_TEMPLATE.md) from the repository root — it has every YAML field pre-filled with comments and a complete content skeleton.
