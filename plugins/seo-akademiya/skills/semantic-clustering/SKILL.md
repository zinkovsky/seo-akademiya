---
name: semantic-clustering
description: Build keyword semantics and cluster it into landing pages via seo-akademiya MCP. Use when the user says "семантика", "кластеризация", "собери ядро", "под какие страницы", "keyword clustering", "структура сайта под запросы".
---

# Semantics → clusters → landing pages

Answer in the user's language. Cost warning: this is the most credit-hungry workflow (1 credit/row on most calls). Estimate and tell the user the budget BEFORE starting: ~1 credit per collected keyword + 10 per cluster-check query.

## Step 1 — collect raw keywords (4 sources)

1. `serpstat_keywords` — seed phrase selection (filter `region_queries_count_from: 30` to cut noise for free).
2. `serpstat_related_keywords` — semantically related (has `weight` = SERP similarity; keep weight ≥ 2).
3. `serpstat_keyword_suggestions` — long-tail, questions.
4. `serpstat_domain_keywords` of 1-2 competitors (from `serpstat_competitors`) — what already works for them.

Dedup (lowercase, trim). For a landing-page task 100-300 keywords is enough — do not pull thousands.

## Step 2 — frequencies in bulk

`serpstat_keywords_info` accepts up to 1000 keywords in ONE call (1 credit each). Drop volume 0. Keep: volume, CPC, difficulty.

## Step 3 — cluster by SERP overlap (the only honest method)

Principle: two keywords belong to one page if Google already ranks the SAME pages for both.

- For each cluster-candidate group take the 1-3 highest-volume keywords and call `serpstat_keyword_full_top` (size 10, 10 credits each). Response field is `top` (not `data`).
- Compare **URLs** between keywords (not domains — in narrow niches the same shops dominate every SERP and domain overlap misleads):
  - ≥ 3 common URLs of 10 → same cluster, one landing page;
  - ≤ 1 common URL → separate pages.
- To economize: first pre-group by shared words/morphology, then verify only borderline pairs with SERP checks. `serpstat_related_keywords` weight ≥ 7 is also a strong same-cluster signal without extra calls.

## Step 4 — map clusters to existing pages

For an existing site: `serpstat_domain_urls` + `gsc_search_analytics` by page — if a cluster's queries already show impressions for some URL, that URL is the landing page; strengthen it instead of creating a duplicate (else you CREATE cannibalization).

## Output format

Table per cluster: **Кластер (головний запит)** | Сумарна частотність | Запити (з частотностями) | Посадкова: існуюча URL або "створити нову" | Пріоритет (частотність ÷ складність).

## Pitfalls

- Mixed-language cores: cluster UA and RU queries separately — SERPs differ.
- Informational vs commercial intent rarely share a page even with SERP overlap — check the top: if blogs rank, it is informational.
- Response of `keyword_full_top` is in `top`, and `se` is REQUIRED for `keyword_top_urls`.
