---
name: seo-analysis
description: Full deep SEO analysis of a project using ALL seo-akademiya MCP sources (GA4, Search Console, Serpstat). Use when the user asks for "SEO анализ", "SEO аудит", "полный анализ сайта", "комплексный анализ", "проанализируй сайт/проект", or wants the complete picture of a site's SEO health.
---

# Full SEO analysis

Answer in the user's language. This is the master workflow: 7 phases, each cross-checks sources against each other. Budget: ~120-250 Serpstat credits (tell the user before starting; `serpstat_credits_stats` is free). GA4/GSC calls are free.

## Phase 0 — resolve & scope

1. `find_project("<name>")` → GA4 property, GSC site, serpstat domain. Note which sources are MISSING — each missing source is a finding for the report.
2. `serpstat_domain_regions_count` (~5 cr) → pick `se` with most keywords. If the site targets several countries, analyze the top-2 databases separately.
3. Situations:
   - **Only a domain, no accesses** (competitor analysis): skip GA4/GSC phases, Serpstat covers market/keywords/backlinks — still a valid analysis, say what's limited.
   - **GSC `siteUnverifiedUser`**: data will be empty — report as access problem.
   - **New site (< 3 months of data)**: shorten periods, note low confidence.

## Phase 1 — market position (Serpstat, ~10 cr)

- `serpstat_domains_info` — visibility, keywords, traffic + dynamics (falling `keywords_dynamic` = losing semantic coverage). Same response carries the **AI Overview block**: `aio_keywords` (how many of our keywords trigger an AI Overview), `aio_citations` (how often WE are cited inside it), `traffic_ai` (estimated AI-affected traffic). Compute citation rate = aio_citations / aio_keywords and compare with competitors (same batch call): they get cited and we don't = losing the AI-SERP layer.
- `serpstat_competitors` (size 10) — keep top-5 true competitors (same business, not marketplaces/wikis). Run `serpstat_domains_info` on them (one batch call, ≤10 domains) → comparison table: visibility, keywords, traffic vs each competitor.

## Phase 2 — search reality (GSC, free)

Window: last 28 full days (exclude last 2 — data lag), plus same window a year ago if the site is old enough.
- by `query` (rowLimit 250): brand vs non-brand split (brand = company/domain name variants). Brand > 60% of clicks = SEO underperforms.
- by `page` (rowLimit 100): traffic concentration — if top-3 pages > 60% of clicks, flag fragility (prostotv case: one page = 40%).
- **Page-2 reserve**: queries with impressions ≥ 500, position 11-20 → fastest wins list.
- **CTR anomalies**: position ≤ 10, impressions ≥ 100, CTR < 60% of expected (p1 30%, p3 10%, p5 6%, p10 2.5%) → snippet problems.
- **AI Overview effect**: GSC does not label AI Overviews separately — impressions/positions include them but clicks vanish. Suspect it when CTR fell at STABLE positions on informational queries; confirm via Serpstat `aio_keywords` (Phase 1) for those query topics.
- **Quick cannibalization check**: `query`+`page` (rowLimit 1000), queries where 2+ pages get impressions ≥ 30; exclude `/ua/` vs `/ru/` hreflang pairs. If found → recommend the `cannibalization` skill for the full workflow.
- by `device` (rowLimit 3): mobile vs desktop — CTR and position gap; mobile position much worse = mobile UX problem.
- by `country` (top 5): where clicks come from; unexpected geo = wrong targeting or spam.
- by `searchAppearance`: which rich results the site earns (product snippets, FAQ, video...) — losing one = CTR drop cause; missing all = structured-data opportunity.
- `dataState: "all"` for the freshest ~48h when investigating something recent.
- year-over-year by `date`: seasonality + trend direction.

## Phase 3 — traffic & money (GA4, free)

- `ga4_run_report` by `sessionDefaultChannelGroup`, two equal windows: channel mix and organic trend. Cross-check: GA4 organic sessions should roughly track GSC clicks — a large gap = tracking problem (check `ga4_data_streams`, `ga4_change_history`; account id from `ga4_property_info` → `property.account`).
- `ga4_key_events`: no key events configured = report cannot tie SEO to business; flag as priority zero.
- If key events exist: `ga4_run_report` metrics `sessions, keyEvents` dimensions `sessionDefaultChannelGroup` → conversion rate of organic vs other channels; then by `landingPage` for organic → which pages convert, not just attract.
- **Engagement quality** of organic landings: metrics `engagementRate, averageSessionDuration, bounceRate` by `landingPage` (organic filter) — pages that attract but do not hold = content/intent mismatch.
- **E-commerce** (if shop): metrics `purchaseRevenue, transactions` by `sessionDefaultChannelGroup` — organic revenue share, the number management cares about; by `landingPage` → money pages.
- by `deviceCategory` and `country` for organic — cross-check with GSC device/geo findings.
- `newVsReturning`: organic bringing only returning users = brand traffic in disguise.
- **12-month trend**: `ga4_run_pivot_report` channels × months (or run_report with `yearMonth`) — direction over a year, not just 28 days.
- `ga4_metadata` first if the property has custom dimensions/metrics (check `ga4_custom_dimensions`) — client may track calls/leads via custom events.
- `ga4_realtime` is NOT part of an audit — skip unless the user asks "что сейчас".

## Phase 4 — semantic coverage (Serpstat, ~20-40 cr)

- `serpstat_domain_keywords` (size 50, filter `region_queries_count_from: 30`): position distribution — % in top-3 / 4-10 / 11-20 / 21+.
- **Keyword gap**: `serpstat_domain_uniq_keywords` vs the strongest competitor (size 30) — what they rank for and we don't; group by topic to propose new pages.
- For 2-3 money pages: `serpstat_url_missing_keywords` (content gap per page) and `serpstat_url_keywords` (what the page already ranks for).
- `serpstat_domain_urls` (size 30): how keywords spread across pages — 1-2 pages holding most keywords = fragile; many zero-keyword pages = index bloat.
- **SERP landscape** for 2-3 top money queries: `serpstat_keyword_full_top` (10 cr each, response in `top`) — who exactly is above us, their SDR/visibility; ads and SERP-feature crowding.
- SERP features per our keywords: `serpstat_domain_keywords` returns them per row — count how often we appear in features vs plain links.

## Phase 5 — backlinks (Serpstat, ~20 cr)

- `serpstat_backlinks_summary` (5 cr) for the site AND top-3 competitors → SDR gap table, referring domains gap.
- `serpstat_backlinks_top_anchors` (free-ish) — anchor profile: brand-heavy = healthy organic; money-anchor-heavy = risky.
- `serpstat_backlinks_threats` (1 cr/row) — toxic links for disavow; "No data found" = clean, good.
- Donor concentration: `serpstat_backlinks_ref_domains` sorted by domain_links desc, size 10 — one donor > 50% of pages = fake-looking profile (upack case: blogspot satellite).
- **Link gap**: `serpstat_backlinks_intersect` with top-2 competitors (1 cr/row, size 20) — donors linking to BOTH competitors but not us = warmest outreach list.
- **Loss dynamics**: `serpstat_backlinks_lost` (size 20) — a burst of recent losses explains ranking drops; note which pages lost links.
- Where links land: `serpstat_backlinks_top_pages` — links only to home = no deep-page authority; compare with which pages NEED authority (money pages from Phase 2).

## Phase 6 — technical health

**Priority order matters**: visibility blockers first (noindex, robots, https, canonical conflicts) — nothing else counts until these are clean; then indexation health; then speed/UX.
- `gsc_list_sitemaps` — errors, last downloaded.
- `gsc_inspect_url` on 3 URLs: home, top category, a page from the losers list — index state, canonical, mobile.
- `serpstat_projects_list` → if a project exists: `serpstat_site_audit_list` + `serpstat_site_audit_basic_info` (free) — SDO score, critical errors; drill into top error category via `serpstat_site_audit_report`. Read `serpstat_site_audit_categories_stat` specifically for: **https** (blockers), **indexation** (noindex/canonical/redirect chains), **pagespeed** (Core Web Vitals proxy — 2026 thresholds: LCP ≤ 2.0s, INP ≤ 200ms, CLS ≤ 0.1), **meta_tags**, **content** (duplicates/thin). If no project/audit → OFFER to create one (MUTATING, 1 cr/page) — never silently.
- `gsc_sitemap_details` on the main sitemap — submitted vs indexed URL counts gap.
- Rank Tracker if present: `serpstat_rt_projects` → `serpstat_rt_project_regions` → `serpstat_rt_url_serp_history` (free) — exact position timeline of tracked keywords, the most precise trend data available; check `serpstat_rt_project_status` first (true = parsing, wait).

## Phase 7 — synthesis (the actual value)

Cross-source findings beat single-source ones. Examples to look for:
- GSC impressions up + GA4 organic flat → ranking for wrong/informational queries.
- Serpstat visibility up + GSC clicks down → gaining low-volume keywords, losing money ones.
- High positions + weak CTR → snippets; high CTR + no conversions (GA4) → landing page/offer.
- Big keyword gap + strong SDR → content problem, not authority; small gap + weak SDR → authority problem.
- Mobile position ≪ desktop (GSC device) + high bounce on mobile (GA4) → mobile UX kills rankings.
- Rich results lost (searchAppearance ↓) + CTR ↓ at same positions → structured data broke; verify with `gsc_inspect_url`.
- CTR ↓ at stable positions + high `aio_keywords` share (Serpstat) → **AI Overview eats the clicks**, not a snippet problem. Response: win citations (concise factual answers, schema, authority) or shift focus to commercial queries where AI Overviews are rare; being cited (`aio_citations`) is the new "position zero".
- Links lost (backlinks_lost) on pages whose positions fell (GSC) → direct cause-effect, prioritize link recovery.
- High organic sessions + near-zero organic revenue (GA4 e-commerce) while other channels convert → SEO lands wrong-intent traffic.

## E-E-A-T & AI-search readiness (proxies via our tools)

E-E-A-T is not a metric, but our data gives honest proxies:
- **Brand strength**: share of brand queries in GSC + brand-anchor share in `serpstat_backlinks_top_anchors` — organic brand demand is the strongest trust signal.
- **Authority**: SDR vs competitors, quality (not count) of referring domains, links from topical sites.
- **AI citations**: `aio_citations` (Phase 1) — being cited by AI Overviews IS the E-E-A-T verdict in practice.
- **Structured data**: `searchAppearance` in GSC (earning rich results = valid schema) + `gsc_inspect_url` rich-results status.
AEO/GEO note: content must be citable — answer-first blocks, clear authorship, dates, schema. If aio_citations ≈ 0 while competitors are cited, recommend a citability review of top pages.

## Beyond our tools — say it, don't fake it

These 2026 audit items are NOT measurable via this MCP; name them in the report as "потребує окремої перевірки" instead of guessing:
- exact Core Web Vitals field data (CrUX / PageSpeed Insights) — we only have Serpstat's pagespeed category as proxy;
- server log analysis (crawl budget, AI-crawler access: GPTBot/ClaudeBot/PerplexityBot in robots.txt);
- schema markup validation beyond what GSC shows;
- content quality/E-E-A-T editorial review (authorship, sources, originality);
- local SEO (Google Business Profile, maps, reviews).

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
