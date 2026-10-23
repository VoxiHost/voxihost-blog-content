# 📂 Source Content (`src/`)

This directory contains the raw Markdown source files and image assets for all VoxiHost guides, tutorials and blog updates. 

## 🗂️ Directory Structure

* **`published/`**: Live posts that are published on the website.
  * Organized by locale: `en/` (English) and `pl/` (Polish), then by section:
    * `guides/` — tutorials and comparisons, published under `/guides/`.
    * `blog/` — release updates and devblogs, published under `/blog/`.
    * `docs/` — product documentation pages, published under `/docs/`. Packaged as `dist/<locale>/docs-<slug>-package.zip`.
  * Inside each section, every post has its own folder named after its URL slug (e.g., `published/en/guides/install-nginx-ubuntu-debian/`).
  * Each post folder contains the main `.md` file and a subfolder named `images/` for any graphics, screenshots, or diagrams.
  * The section is determined by the `category` frontmatter: `Tutorials` / `Comparisons` → `guides/`, `Updates` → `blog/` (and the Polish equivalents). Docs pages have no category: their folder under `docs/` is what routes them.
* **`drafts/`**: In-progress drafts, guest article suggestions, and posts awaiting review or scheduled for release.
  * Organized similarly by locale (`en/`, `pl/`) and slug folder.
  * *Note:* Unlike published articles, drafts **do not** compile to ZIP files in the `dist/` directory.

## ✍️ How to Add or Edit an Article

1. **For Edits:** Locate the article's folder under `published/<locale>/<guides|blog>/<slug>/`, make your changes to the `.md` file, and commit.
2. **For New Articles:**
   * Create a folder under `drafts/<locale>/<slug-name>/`.
   * Create a Markdown file inside it named `<slug-name>.md`.
   * Fill out the required YAML frontmatter (with `status: draft`).
   * Put any screenshots/graphics inside a subfolder named `images/`.

For complete contribution requirements, formatting guidelines, and licensing terms, please refer to [CONTRIBUTING.md](../CONTRIBUTING.md) in the repository root.

> **New article?** Start by copying [ARTICLE_TEMPLATE.md](../ARTICLE_TEMPLATE.md) from the repository root — it has every YAML field pre-filled with comments and a complete content skeleton.
