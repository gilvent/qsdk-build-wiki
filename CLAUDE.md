# QSDK Build Wiki

A wiki about using QSDK source code to build OpenWrt firmware or package. Pages may include contents for specific target board or general concepts and fundamentals. Three layers:

- `references/` — raw sources (PDFs, papers). Immutable: read, never edit. Binaries are gitignored; their names/provenance are tracked in `references/README.md`.
- `wiki/` — generated markdown pages: summaries, comparisons, syntheses. You own this layer.
- This file — structure and workflow.

## Operations

**Query** — User asks a question. Answer from existing `wiki/` pages, citing them. If the
question is substantial and reusable, write or update a page, then update `wiki/index.md`
and append a `QUERY` entry to `wiki/log.md`. Ground every claim in `references/`; if no
source covers it, say so rather than inventing.

**Lint** — Validate wiki health, fix what is safe, flag what is not, and append a `LINT`
entry to `wiki/log.md`. Checks:
- frontmatter present and complete (`title`, `tags`, `sources`, `updated`); H1 matches
  `title`; a one-line summary follows the H1
- every page is linked from `wiki/index.md`; no orphans
- every `references/` source is listed in `references/README.md` (binaries are gitignored); external URLs are allowed
- internal links resolve
- contradictions and stale claims (flag only, don't auto-resolve)

No automated Ingest — pages are created on demand from a Query.

Pages synthesized across many sources (comparisons, deep-research reports, etc.) are normal
`wiki/` pages — file them under an `index.md` category. Keep `wiki/` flat; don't create
per-type subfolders until a category is genuinely unwieldy.

## Page structure

File name: kebab-case, topic first (e.g. `80211r-fast-bss-transition.md`); date-prefix
dated reports (e.g. `2026-06-08-roaming-kpi-comparison.md`). Generated images/charts go in
`wiki/assets/` (create it when the first one appears).

```yaml
---
title: <page title>
tags: [<kebab-tags>]                  # freeform; not yet used by tooling
sources: [references/<file> | <url>, ...]   # references/ files and/or external URLs
updated: <YYYY-MM-DD>                 # bump on any content change
---
```

After the frontmatter: an H1 matching `title`, a one-line *italic* summary, then a
`> Sources:` line naming where in those files the page draws from (clauses, sections,
pages). In the body, cite specific claims inline (`§13.5`, `p. 3150`). Link related pages
with relative links.

Provenance lives in exactly two places: frontmatter `sources` (which files or URLs) and
citations of where in them — the `> Sources:` line plus inline references. Don't add other
kinds of source notes.

## index.md

Registry of all pages, grouped under freeform H2 categories. One line per page: relative
link + one-line summary. Lint treats it as the authoritative page list.

## log.md

Append-only, newest at the bottom. One entry per operation:

```
## [YYYY-MM-DD] OP | <question or task>
- Pages: <created/updated paths>
- Sources: <references used, with section/page>
- Notes: <optional>
```

`OP` ∈ {`QUERY`, `LINT`, `EDIT`}.
