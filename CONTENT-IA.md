# Content IA — what goes where

VoxiHost publishes four kinds of content. Each has one home, one URL namespace and
one job. This keeps the site sections from competing with each other in search and
navigation — the same split DigitalOcean, Vultr and OVHcloud use between reference
docs, tutorials and announcements.

## The four types

| Type | Where it lives | Answers |
|---|---|---|
| **Product docs** | `voxihost.pl/docs/` — maintained by the VoxiHost team | "What does this feature do? What are the limits? Where is it in the panel?" |
| **Guides** | This repo → `src/published/<locale>/guides/<slug>/` → `/guides/` | "How do I install / configure / fix X, end to end?" |
| **Updates / blog** | This repo → `src/published/<locale>/blog/<slug>/` → `/blog/` | "What changed and when?" |
| **FAQ** | `voxihost.pl/faq/` — maintained by the VoxiHost team | "Why is my account / bill / server behaving like this?" (one short answer) |

**Which section does my article go to?** The `category` frontmatter decides it:

- `Updates` / `Nowości` → `/blog/`
- everything else (`Tutorials`, `Poradniki`, `Comparisons`, `Porównania`, …) → `/guides/`

Everything you write in this repo is either a guide or an update. Docs and FAQ are
written by the team in the website repository.

## One intent, one page

- **A procedure with prerequisites and steps → guide.** That is what this repo is for.
- **An announcement → update.** It is not maintained for accuracy after publishing: it links to the current guide or docs page instead. If a release note is the only place a feature is documented, that is a docs gap — tell the team.
- **Reference material** (limits, flags, billing rules, panel behaviour) does not belong in a guide. A guide may *summarise* it in a sentence and link to the docs.
- **A policy or "why" answer that fits in a paragraph** belongs in the FAQ, not in a guide.

Before writing, check whether a page for that intent already exists. If it does,
update it instead of adding a second one.

## Guides vs updates, in practice

| | Guide | Update |
|---|---|---|
| Lifetime | Updated as software changes | Frozen once published |
| Structure | Title → intro → prerequisites → steps → conclusion | Title → what changed → links |
| Frontmatter | `category` = tutorial-style value | `category: Updates` / `Nowości` |
| Reviews | Accuracy is re-checked over time | Only factual corrections, never rewrites |
| Redirects | Old slug → new slug when replaced | Never renamed |

## Cross-linking

Every article should link one level deeper than itself rather than repeat what is
already written:

- **Guide → product docs** for the reference details of the feature it configures.
- **Update → the guide and the docs page** for what it announces.
- **FAQ → the exact doc or guide page** that explains the answer in full.
- **Related guides**: link 2–3 genuinely related guides, not the same three every time.

## Lifecycle

- When a guide is superseded, publish the replacement and tell the team so the old
  URL can be redirected (301). Do not silently rewrite a guide into a different
  topic under the same URL.
- A guide that has become mostly reference should be slimmed to the task; the
  reference moves to the docs.
- When two pages explain the same thing, the docs page is canonical and the other
  page links to it.

## Before publishing, ask

1. Could this live as a section of an existing page? Then do that.
2. Is there already a page for this intent? Update it, don't add a second.
3. Does it link to the page one level deeper (guide → docs, update → guide + docs)?
4. Does it duplicate a docs paragraph? Cut the paragraph and link instead.
5. Does the `category` frontmatter send it to the section you actually intend?
