---
name: meta-tags
description: Generate title/description/H1 from semantics and real GSC query data via seo-akademiya MCP. Use when the user says "метатеги", "мета-теги", "тайтлы", "title и description", "пропиши мету", "СTR сниппетов".
---

# Meta tags from semantics + GSC reality

Answer in the user's language. Meta tags are written per PAGE, from the page's cluster + what it ALREADY shows for in Google.

## Step 1 — evidence per page

For each target page:
1. `gsc_search_analytics` filtered by page (`dimensionFilterGroups` with `page equals <url>`), by `query`, rowLimit 25 — real queries, positions, CTR.
2. `serpstat_url_keywords` (1 credit/row) — fuller keyword list with volumes for pages weak in GSC.
3. If a semantic cluster exists from the semantic-clustering skill — use it as the primary source; GSC confirms wording users actually type.

## Step 2 — prioritize by CTR loss

Expected CTR by position (rough): p1 30%, p2 16%, p3 10%, p4 7%, p5 6%, p6-10 5→2.5%.
Flag queries with impressions ≥ 100 where actual CTR < 60% of expected — these pages lose clicks at CURRENT positions; meta rewrite gives traffic without ranking growth. Pages where CTR is fine or above expected: do NOT touch working titles — say so explicitly.

## Step 3 — write the tags

Rules:
- **Title ≤ 60 chars**: main query (exact form users type, from GSC) at the start → key qualifier (купити/ціна/city) → brand at the end after "—" or "|". No keyword stuffing, no ALL CAPS.
- **Description 140-160 chars**: main + 1-2 secondary queries naturally, one concrete benefit (доставка, гарантія, ціна від...), call to action. It is an ad, not a keyword list.
- **H1**: main query or its natural variant, ≠ title verbatim, no brand.
- One language per page version (/ua/ pages get Ukrainian meta, /ru/ Russian).
- Every secondary query used must come from the page's own cluster — never borrow from a neighboring page's cluster (creates cannibalization).

## Output format

Table: **URL** | Головний запит (частотність) | Title (з довжиною в дужках) | Description (довжина) | H1 | Причина зміни (CTR-втрата / нова семантика / немає мети).

## Pitfalls

- GSC shows the query in the user's spelling ("очки рейбен") — use the highest-volume spelling in the title, cover variants in description.
- If CTR ≥ expected everywhere, meta is not the problem — recommend content/position work instead of rewriting for the sake of it.
