# Benchmark Results — Iteration 1

**Skill:** gsc-intent-gap-finder
**Date:** 2026-06-18
**Evals run:** eval-1 (breakingb2b full analysis), eval-2 (samdunning positions 10-20)
**Configurations:** with_skill vs. without_skill (no skill, Ahrefs MCP available to both)

---

## Summary

| Configuration | Pass Rate | Avg Tokens | Avg Time |
|---|---|---|---|
| **with_skill** | **82.8%** | 113k | 504s |
| without_skill | 70.7% | 99k | 460s |
| **Delta** | **+12.1%** | +14k | +44s |

---

## Per-eval breakdown

### Eval 1 — breakingb2b.com full intent gap analysis

| | Pass Rate | Passed | Failed | Tokens | Time |
|---|---|---|---|---|---|
| with_skill | 80% | 8/10 | 2/10 | 112,024 | 544.9s |
| without_skill | 70% | 7/10 | 3/10 | 101,346 | 545.3s |

**Assertions failed by both:** GSC tools not invoked (GSC MCP authenticated to a different property — risevision.com); Global MSV labelling not used.

**What with_skill added:** Explicit "What Google rewards" SERP evidence sections per gap; supporting keyword clusters per opportunity; tiered prioritization (Tier 1/2/3); three structural patterns meta-analysis section.

**What without_skill did better:** Found 18 gaps vs 10; included full appendix table of all 66 keywords; spotted zero-traffic #1 rankings (3 pages ranking #1 with 12,700 combined monthly searches getting 0 clicks — highest-ROI finding in the analysis).

---

### Eval 2 — samdunning.org positions 10-20

| | Pass Rate | Passed | Failed | Tokens | Time |
|---|---|---|---|---|---|
| with_skill | 85.7% | 6/7 | 1/7 | 114,795 | 463.9s |
| without_skill | 71.4% | 5/7 | 2/7 | 96,059 | 375.4s |

**Assertion failed by both:** GSC data could not be pulled for samdunning.org (no organic keywords — personal site, not the content site). Both agents correctly identified breakingb2b.com as the real content domain.

**What with_skill added:** Full keyword table for all 13 in-range keywords; "What #1 looks like" SERP sections; supporting keyword clusters; "new page vs update" field per opportunity.

**What without_skill missed:** No supporting keyword clusters; simpler structure; found 5 gaps vs 6.

---

## Iteration 1 improvements applied

Based on these results, the skill was updated to:

1. **Accept GSC CSV export as primary data source** — user exports from Search Console and uploads. Avoids MCP authentication issues entirely.
2. **Make Ahrefs the primary path** — GSC MCP is supplemental. Ahrefs `site-explorer-organic-keywords` covers positions 1–50 without authentication issues.
3. **Added zero-traffic rankings section** — explicitly flag pages ranking top 5 with near-zero traffic (CTR/AI Overview issue). Highest-ROI quick win category.
4. **Expanded position range to 1–50** — original 4–30 was too narrow; useful gaps found at 31–45.
5. **Strengthened exhaustiveness nudge** — skill now explicitly says to work through all candidates, not stop at representative examples.
6. **Global MSV labelling** — added reminder in both the research phase and the output format section.
