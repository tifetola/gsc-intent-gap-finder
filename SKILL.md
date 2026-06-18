---
name: gsc-intent-gap-finder
description: >
  Pull every keyword from Google Search Console (via CSV export or MCP), cross-reference
  SERP format expectations via Ahrefs, crawl each ranking page, and surface a prioritised
  list of search intents where no existing page fully delivers — revealing net-new content
  creation opportunities. Trigger whenever the user wants to find new content to create
  from GSC data, identify intent mismatches on existing pages, or turn rankings into a
  content roadmap. Always trigger when the user says "what should I write based on my
  rankings", "what content am I missing", "what keywords do I rank for without good pages",
  "GSC content gaps", "intent gaps", "do an intent gap analysis", or anything about finding
  content opportunities from ranking data. Also trigger when the user uploads a GSC
  performance CSV or mentions a search console export and wants content recommendations.
compatibility: "Requires Ahrefs MCP. GSC data via uploaded CSV export or GSC MCP (if authenticated to the right property). Web fetch or Claude in Chrome for page crawling."
---

# GSC Intent Gap Finder

Surfaces net-new content creation opportunities by combining Google Search Console ranking data with Ahrefs SERP intelligence and live page crawls. The output is a prioritised list of search intents where you rank but don't have a page that fully delivers — each one a concrete, evidenced case for creating new content.

## The core insight

When you rank at position 5–50 for a keyword, Google has decided you're relevant — but not the best answer. That's a signal, not a failure. It means a new, intent-matched page could break into the top 3 much faster than starting from zero.

This skill finds every instance of that gap across your entire keyword set.

---

## Phase 1 — Get the keyword data

### Primary: GSC CSV export (preferred)

Ask the user to export from Google Search Console:
- **Search Console → Performance → Pages** tab, then switch to **Queries** 
- Export as CSV (top-right download button)
- Default is 90 days / 1,000 rows — for deeper analysis, extend to 16 months if available

The CSV will have columns: `Top queries`, `Clicks`, `Impressions`, `CTR`, `Position`. Read it with Bash:

```bash
# The file will be in /mnt/user-data/uploads/ if uploaded, or wherever the user specifies
head -5 /path/to/gsc-export.csv
wc -l /path/to/gsc-export.csv
```

If the user also has the **Pages** export (same Performance report, Pages tab), that's even better — it gives you the landing URL per keyword. Ask for both if convenient.

### Fallback: GSC MCP

If the GSC MCP is authenticated to the right property, use:
1. `advanced_search_analytics` — pull queries with impressions, clicks, CTR, position (limit to position ≤ 50, impressions ≥ threshold)
2. `content_gaps` — run this directly; it may surface intent gaps automatically
3. `ctr_opportunities` — keywords with below-benchmark CTR (often signals intent mismatch)
4. `cannibalization_check` — pages competing for the same queries

If the GSC MCP is not authenticated to the right property, proceed with Ahrefs alone — it's a complete data source.

### Always: Ahrefs site-level data

Run these regardless of whether you have a GSC CSV:

1. `site-explorer-organic-keywords` — all ranking keywords with position, volume, KD, URL. Positions 1–50.
2. `site-explorer-top-pages` — which pages drive traffic and what keyword clusters they own
3. `site-explorer-metrics` — DR, organic traffic, keyword count baseline

> If any Ahrefs tool returns `render_with` in its metadata, call that render tool immediately on the returned data.

---

## Phase 2 — Set scope

Confirm with the user (or infer from context):

- **Domain**: e.g. `breakingb2b.com`
- **Branded terms to exclude**: your brand name and common branded phrases — filter these out before analysis
- **Minimum impressions threshold**: default 50/month; for niche B2B, try 20
- **Any focus areas**: e.g. "only /blog/", "skip competitor comparison pages" — proceed without constraints if not specified

---

## Phase 3 — Build the keyword-to-page map

Merge your GSC data with Ahrefs data into one working dataset:

| Keyword | GSC Position | GSC Impressions | GSC Clicks | GSC CTR | Ahrefs Global Volume | KD | Landing URL |

- GSC is authoritative for actual clicks and impressions
- Ahrefs fills in **global search volume** (use `global_volume` field — not US-only `volume`) and KD
- Where a keyword appears in both sources, keep both data points

Group by `Landing URL`. For each URL group, note:
- **Position spread**: all keywords landing near position 1–3, or a wide range with some at 30+?
- **Intent diversity**: do all keywords want the same thing, or is there a mix of informational, commercial, transactional?
- **Volume-to-click gap**: high impressions but near-zero clicks at good positions = something is misaligned

Flag URL groups showing wide position spreads, mixed intents, or poor CTR at decent positions. These are your candidates for deeper analysis.

---

## Phase 4 — Identify intent mismatches

Work through all candidate keywords exhaustively — don't stop after finding a representative sample. Every gap matters.

For each candidate keyword, check for these five signals:

### Signal A — SERP format mismatch (most common)
Use `serp-overview` (Ahrefs) on the keyword. Check positions 1–5:

- Top 5 are long-form guides but your page is a product/service page → **depth/format gap**
- Top 5 are listicles ("Best X for Y") but you have a category or overview page → **listicle gap**
- Top 5 include comparison pages ("X vs Y") but you don't have one → **comparison gap**
- Top 5 are informational how-tos but you're serving a commercial page → **intent type mismatch**
- Top 5 are editorial roundups but your page is self-promotional → **credibility format gap**

This is the most reliable signal. When Google consistently returns a different content format to what you have, a new page matching that format has a structural advantage.

### Signal B — Mixed intents on one page
A single URL ranking for both "what is X" (informational) and "buy X" (transactional) means neither group is served well. Flag as: **split-intent page**.

### Signal C — Low CTR at positions 3–10
Below-expected CTR at good positions often means the page title/format doesn't match what the searcher actually wants. Sometimes fixable with a rewrite; sometimes the query needs a fundamentally different page.

### Signal D — Position 11–50 keywords with real volume
Ranking footholds you haven't converted. Check: is the landing page primarily built for this query, or is it ranking incidentally? If incidental, there's a case for a dedicated page.

### Signal E — Cannibalization
Where multiple pages compete for the same query, the fix is consolidation — one definitive page absorbing all the authority.

---

## Phase 5 — Crawl the ranking pages

For each URL in your candidate set, fetch the live page. Use `web_fetch` first; fall back to Claude in Chrome if the page is JavaScript-rendered.

Extract:
- **Title tag and H1** — do they match the keyword cluster intent?
- **Content type** — listicle, how-to guide, product landing page, case study, comparison, FAQ, etc.
- **Depth and coverage** — skim H2s/H3s: what questions does this page answer? What would a searcher with this intent expect to find that isn't there?
- **Conversion orientation** — is the page driving toward a demo/purchase when the searcher is in learning mode?

You're not judging quality — you're asking: does this page satisfy the intent of the keywords landing on it?

---

## Phase 6 — Validate with keyword research

For each confirmed gap, validate before recommending:

1. `keywords-explorer-overview` — **global_volume** (label as **Global MSV** in the output, never US-only), KD, CPC, traffic potential
2. `keywords-explorer-matching-terms` or `keywords-explorer-related-terms` — find supporting keywords to build the full cluster

**Qualification thresholds** (adjust to site DR and niche):
- Global MSV ≥ 50 (or ≥ 20 for niche B2B)
- KD ≤ DR + 10 as a rough guide
- Business relevance: would someone searching this term plausibly engage with the brand?

Drop anything that fails. Don't pad the output with unqualified targets.

---

## Phase 7 — Write the Content Opportunity Report

Write directly in chat. Use `render-data-table` from Ahrefs when available.

---

### [Domain] — Intent Gap & Content Opportunity Report

**Summary** (2–3 sentences): how many gaps found, dominant gap types, highest-priority cluster.

**Site baseline**: DR X | ~X organic keywords | ~X organic visits/mo

---

### Priority Content Opportunities

| # | Content to Build | Primary Keyword | Global MSV | KD | Gap Type | Current Ranking URL | Position |
|---|---|---|---|---|---|---|---|

Sort by: volume × (1 - KD/100) descending.

---

### Detailed breakdown — top opportunities

For each opportunity:

**[Working title for the content]**
- **Primary keyword**: `keyword` — Global MSV: X | KD: X
- **Supporting keywords**: keyword1 (X), keyword2 (X), keyword3 (X)
- **Gap evidence**: what you found in the SERP and on the current page. Be specific: "Top 5 results are all 2,000+ word how-to guides. The ranking page (/services/x) is a 400-word product description with no instructional content."
- **What to build**: content type, format, approximate depth, key sections to include
- **Suggested slug**: `/blog/[slug]`
- **New page vs. update**: new page / expand existing / consolidate and redirect

---

### Zero-traffic rankings — separate flag

Identify any pages ranking **position 1–5 with zero or near-zero traffic**. This is a distinct problem from intent gaps — the page is winning the SERP but not winning clicks. Likely causes: AI Overview absorbing clicks, title/meta mismatch, SERP features stealing visibility. Flag these explicitly:

| Page | Keyword | Global MSV | Position | Est. Traffic | Likely cause |
|---|---|---|---|---|---|

These represent the highest-ROI quick wins on the site — a title/meta fix on a #1 ranking page can unlock hundreds of clicks with zero new content.

---

### Cannibalization fixes

URL pairs or groups to consolidate. Specify which URL becomes canonical and why.

---

### Quick wins (existing pages, no new content needed)

Cases where intent can be addressed by improving an existing page: title rewrites, adding an FAQ section, restructuring H2s, expanding a thin section. These are faster than new pages.

---

### Pages with correct intent alignment (do not disturb)

Brief reference table of pages that are well-matched and performing — so the user knows what not to touch.

---

## Notes

- **Work through all candidates** — don't stop at 5-10 representative examples. Every gap is a content opportunity. A comprehensive analysis is more useful than a curated one.
- **Always crawl before calling something a gap** — pages sometimes are better than their position suggests
- **New page vs. update**: fundamentally wrong content type → new page. Right type but thin/structured badly → update
- **Volume labelling**: always use "Global MSV" — never US-only volume without flagging it
- **Branded queries**: almost never indicate a content gap — exclude them throughout
- **AI Overviews**: if the SERP has an AI Overview, educational content that can be cited is more valuable than ranking position alone

---

## Tool reference

| Task | Primary tool | Fallback |
|---|---|---|
| GSC keyword data | CSV export (uploaded file) | `advanced_search_analytics`, `gsc-keywords` |
| GSC content gap detection | `content_gaps`, `content_recommendations` | Manual analysis |
| CTR analysis | `ctr_opportunities`, `ctr_vs_benchmark` | GSC raw data |
| Cannibalization | `cannibalization_check` | Manual grouping |
| SERP format check | `serp-overview` | — |
| Keyword volume + KD | `keywords-explorer-overview` | `gsc-keywords` |
| Supporting keyword cluster | `keywords-explorer-matching-terms` | `keywords-explorer-related-terms` |
| Site organic profile | `site-explorer-organic-keywords`, `site-explorer-top-pages`, `site-explorer-metrics` | — |
| Page crawl | `web_fetch` | Claude in Chrome |
| Render tables | `render-data-table` | Markdown |
