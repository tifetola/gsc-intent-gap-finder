# gsc-intent-gap-finder

Claude Code skill: GSC intent gap analysis — find keywords you rank for but don't have pages fully satisfying the search intent, surfacing net-new content creation opportunities.

## What it does

1. Pulls your full keyword set from Google Search Console (via CSV export or MCP) and Ahrefs
2. Groups keywords by landing URL and flags intent mismatches
3. Runs SERP analysis via Ahrefs to check what content format Google actually rewards for each query
4. Crawls your current ranking pages to assess how well they match the intent
5. Validates opportunities with keyword volume and KD data
6. Produces a prioritised content opportunity report with SERP evidence, supporting keyword clusters, and suggested content formats

## Gap signals it detects

- **SERP format mismatch** — you have a service page but Google ranks guides; you have a blog post but Google ranks listicles
- **Split intent** — one page ranking for both informational and commercial queries
- **Incidental rankings** — pages ranking position 11–50 for queries they weren't built for
- **Cannibalization** — multiple pages splitting authority on the same query
- **Zero-traffic #1 rankings** — pages ranking top 5 but getting no clicks (AI Overview / title mismatch)

## How to use

1. Export your Google Search Console performance data: **Search Console → Performance → Queries → Export CSV**
2. Upload the CSV and trigger the skill: *"Do an intent gap analysis on [domain] based on my GSC export"*
3. Or just trigger it by domain without a CSV — the skill will use Ahrefs data directly

## Requirements

- [Ahrefs MCP](https://ahrefs.com) connected in Claude Code
- GSC CSV export (recommended) or GSC MCP authenticated to the right property

## Install

Download `gsc-intent-gap-finder.skill` and drag it into Claude Code, or install via the skills panel.
