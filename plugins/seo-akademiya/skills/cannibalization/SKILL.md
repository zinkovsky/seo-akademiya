---
name: cannibalization
description: Find and fix keyword cannibalization (several pages competing for one query) via seo-akademiya MCP. Use when the user says "каннибализация", "канібалізація", "страницы конкурируют", "две страницы по одному запросу", "cannibalization".
---

# Keyword cannibalization

Answer in the user's language. Cannibalization = two+ pages of the SAME site getting impressions for the SAME query, splitting relevance and losing to focused competitors.

## Step 1 — detect (GSC, free)

`gsc_search_analytics` with `dimensions: ["query","page"]`, rowLimit 1000+, 4-week window. Group rows by query; keep queries where ≥ 2 pages have impressions ≥ 30.

**Filter out false positives first:**
- language versions: `/ua/x` vs `/x` or `/ru/x` — hreflang pair, NOT cannibalization; skip (unless positions are far apart, then hreflang may be broken — note separately);
- brand queries hitting home + contacts + about — normal;
- pagination and near-identical parametrized URLs.

## Step 2 — confirm and measure harm

For each real case:
- Which page does GOOGLE prefer (more impressions, better position)? Does it match which page the BUSINESS prefers (category vs old blog post)?
- `serpstat_domain_keywords` with the query in filters — Serpstat's view of which URL ranks (1 credit/row, optional confirmation).
- Harm signal: combined position worse than the best page's historical position; position jumping between weeks (Google swapping URLs).

## Step 3 — prescribe (one of four, per case)

| Situation | Fix |
|---|---|
| Old/thin page vs main category | **301 redirect** old → main, move unique content first |
| Both pages valuable, different intent shades | **De-optimize** the loser: remove query from its title/H1, re-anchor internal links to the winner |
| Near-duplicates (filters, utm-clones) | **canonical** to the winner |
| Same intent, both weak | **Merge** content into one URL, 301 the other |

Internal linking always: all mentions of the query link to the chosen winner.

## Output format

Table: **Запит** (покази) | Сторінки-суперники (покази, позиція кожної) | Переможець | Рецепт (301 / деоптимізація / canonical / злиття) | Очікуваний ефект.

Sort by total impressions desc — fix the biggest first.

## Pitfalls

- GSC rowLimit caps at 25000; for large sites paginate with `startRow` or narrow with query filters.
- Do not prescribe 301 for pages that earn conversions in GA4 — check `ga4_run_report` by pagePath for the loser page before recommending deletion.
- Re-check 3-4 weeks after fixes: same GSC query+page report, cases should disappear.
