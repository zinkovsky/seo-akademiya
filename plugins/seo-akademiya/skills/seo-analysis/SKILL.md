---
name: seo-analysis
description: Full deep SEO analysis of a project using ALL seo-akademiya MCP sources (GA4, Search Console, Serpstat). Use when the user asks for "SEO анализ", "SEO аудит", "полный анализ сайта", "комплексный анализ", "проанализируй сайт/проект", or wants the complete picture of a site's SEO health.
---

# Full SEO analysis

Answer in the user's language. This is the master workflow: 7 phases, each cross-checks sources against each other. Budget: ~80-150 Serpstat credits (tell the user before starting; `serpstat_credits_stats` is free). GA4/GSC calls are free.

## Phase 0 — resolve & scope

1. `find_project("<name>")` → GA4 property, GSC site, serpstat domain. Note which sources are MISSING — each missing source is a finding for the report.
2. `serpstat_domain_regions_count` (~5 cr) → pick `se` with most keywords. If the site targets several countries, analyze the top-2 databases separately.
3. Situations:
   - **Only a domain, no accesses** (competitor analysis): skip GA4/GSC phases, Serpstat covers market/keywords/backlinks — still a valid analysis, say what's limited.
   - **GSC `siteUnverifiedUser`**: data will be empty — report as access problem.
   - **New site (< 3 months of data)**: shorten periods, note low confidence.

## Phase 1 — market position (Serpstat, ~10 cr)

- `serpstat_domains_info` — visibility, keywords, traffic + dynamics (falling `keywords_dynamic` = losing semantic coverage).
- `serpstat_competitors` (size 10) — keep top-5 true competitors (same business, not marketplaces/wikis). Run `serpstat_domains_info` on them (one batch call, ≤10 domains) → comparison table: visibility, keywords, traffic vs each competitor.

## Phase 2 — search reality (GSC, free)

Window: last 28 full days (exclude last 2 — data lag), plus same window a year ago if the site is old enough.
- by `query` (rowLimit 250): brand vs non-brand split (brand = company/domain name variants). Brand > 60% of clicks = SEO underperforms.
- by `page` (rowLimit 100): traffic concentration — if top-3 pages > 60% of clicks, flag fragility (prostotv case: one page = 40%).
- **Page-2 reserve**: queries with impressions ≥ 500, position 11-20 → fastest wins list.
- **CTR anomalies**: position ≤ 10, impressions ≥ 100, CTR < 60% of expected (p1 30%, p3 10%, p5 6%, p10 2.5%) → snippet problems.
- **Quick cannibalization check**: `query`+`page` (rowLimit 1000), queries where 2+ pages get impressions ≥ 30; exclude `/ua/` vs `/ru/` hreflang pairs. If found → recommend the `cannibalization` skill for the full workflow.
- year-over-year by `date`: seasonality + trend direction.

## Phase 3 — traffic & money (GA4, free)

- `ga4_run_report` by `sessionDefaultChannelGroup`, two equal windows: channel mix and organic trend. Cross-check: GA4 organic sessions should roughly track GSC clicks — a large gap = tracking problem (check `ga4_data_streams`, `ga4_change_history`; account id from `ga4_property_info` → `property.account`).
- `ga4_key_events`: no key events configured = report cannot tie SEO to business; flag as priority zero.
- If key events exist: `ga4_run_report` metrics `sessions, keyEvents` dimensions `sessionDefaultChannelGroup` → conversion rate of organic vs other channels; then by `landingPage` for organic → which pages convert, not just attract.

## Phase 4 — semantic coverage (Serpstat, ~20-40 cr)

- `serpstat_domain_keywords` (size 50, filter `region_queries_count_from: 30`): position distribution — % in top-3 / 4-10 / 11-20 / 21+.
- **Keyword gap**: `serpstat_domain_uniq_keywords` vs the strongest competitor (size 30) — what they rank for and we don't; group by topic to propose new pages.
- For 2-3 money pages: `serpstat_url_missing_keywords` (content gap per page).

## Phase 5 — backlinks (Serpstat, ~20 cr)

- `serpstat_backlinks_summary` (5 cr) for the site AND top-3 competitors → SDR gap table, referring domains gap.
- `serpstat_backlinks_top_anchors` (free-ish) — anchor profile: brand-heavy = healthy organic; money-anchor-heavy = risky.
- `serpstat_backlinks_threats` (1 cr/row) — toxic links for disavow; "No data found" = clean, good.
- Donor concentration: `serpstat_backlinks_ref_domains` sorted by domain_links desc, size 10 — one donor > 50% of pages = fake-looking profile (upack case: blogspot satellite).

## Phase 6 — technical health

- `gsc_list_sitemaps` — errors, last downloaded.
- `gsc_inspect_url` on 3 URLs: home, top category, a page from the losers list — index state, canonical, mobile.
- `serpstat_projects_list` → if a project exists: `serpstat_site_audit_list` + `serpstat_site_audit_basic_info` (free) — SDO score, critical errors; drill into top error category via `serpstat_site_audit_report`. If no project/audit → OFFER to create one (MUTATING, 1 cr/page) — never silently.
- Rank Tracker if present: `serpstat_rt_projects` → position trends for free.

## Phase 7 — synthesis (the actual value)

Cross-source findings beat single-source ones. Examples to look for:
- GSC impressions up + GA4 organic flat → ranking for wrong/informational queries.
- Serpstat visibility up + GSC clicks down → gaining low-volume keywords, losing money ones.
- High positions + weak CTR → snippets; high CTR + no conversions (GA4) → landing page/offer.
- Big keyword gap + strong SDR → content problem, not authority; small gap + weak SDR → authority problem.

## Output format

1. **Executive summary** — 3-5 sentences: overall health, the ONE biggest lever.
2. **Scorecard table**: Ринок | Семантика | Контент/CTR | Посилання | Техстан | Конверсії — оцінка ✅/⚠️/🔴 + один рядок доказу.
3. **Деталі по фазах** — тільки знахідки, без сирих даних.
4. **План дій** — max 7 пунктів, кожен: що зробити → на основі якого факту → очікуваний ефект. Сортувати за (ефект ÷ зусилля).

## Pitfalls

- Do NOT call `serpstat_url_summary_traffic` (1000 cr) or start audits/scans without explicit user confirmation.
- Identical repeated calls are cached 24h and free — re-running the analysis same day costs almost nothing.
- If credits `left_lines` < 2000, warn and offer the free-only version (GSC+GA4 phases).
- Rate limit 1 rps: full analysis takes several minutes — tell the user.
