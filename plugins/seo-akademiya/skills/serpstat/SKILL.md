---
name: serpstat
description: Expert guidance for Serpstat SEO analysis via seo-akademiya MCP tools (serpstat_*). Use for keyword research, competitor analysis, backlinks audit, content gaps, rank tracking, site audits, or whenever the user mentions Serpstat, keywords, backlinks, SERP positions, or SEO analysis of any domain.
---

# Serpstat analysis via seo-akademiya

65 `serpstat_*` tools cover the full Serpstat API v4. Answer in the user's language.

## Golden rules

1. **Credits are shared across the team.** Before any workflow that will pull many rows, call `serpstat_credits_stats` (free, cached 5 min). If `left_lines` < 1000, warn the user before proceeding. Most analytics tools cost 1 credit per returned row; responses include `summary_info.left_lines` (remaining credits).
2. **Pick the right `se` first.** For an unfamiliar domain call `serpstat_domain_regions_count` (~5 credits) and use the database with the most keywords. Common codes: `g_ua`, `g_us`, `g_uk`, `g_de`, `g_pl`, `bing_us`. Ukrainian businesses are usually `g_ua`.
3. **Keep sizes small.** Defaults are already conservative (50). Ask for more only when the user needs it. Filters like `region_queries_count_from: 100` cut low-volume noise for free.
4. **Repeated identical calls are free** — the server caches analytics for 24h (`cached: true` in the response), so re-querying the same data costs nothing.
5. **Expensive tools — confirm with the user first:**
   - `serpstat_url_summary_traffic` — **1000 credits per data type**
   - `serpstat_site_audit_start` — 1 credit/page (10 with JS rendering); check `pagesLimit` in settings first
   - `serpstat_page_audit_scan` / `serpstat_page_audit_rescan` — 10 credits each
6. **MUTATING tools** (create/delete project, start/stop audits, set settings) — never call without an explicit user request; `serpstat_project_delete` and `serpstat_page_audit_remove` are irreversible, always confirm.
7. Rate limit is 1 request/second (the server queues automatically) — long multi-call workflows take time; tell the user when a workflow needs 10+ calls.

## Tool map

- **Domain**: `serpstat_domains_info` (bulk overview, ≤10), `serpstat_competitors`, `serpstat_domain_keywords`, `serpstat_domain_urls`, `serpstat_domain_regions_count`, `serpstat_domain_uniq_keywords` (keyword gap), `serpstat_market_categories` → `serpstat_category_top_domains`
- **Keywords**: `serpstat_keywords` (selection), `serpstat_related_keywords`, `serpstat_keywords_info` (bulk ≤1000), `serpstat_keyword_suggestions` (long-tail), `serpstat_keyword_full_top` (SERP + domain metrics), `serpstat_keyword_top_urls`, `serpstat_keyword_competitors`
- **URL**: `serpstat_url_keywords`, `serpstat_url_competitors`, `serpstat_url_missing_keywords` (content gap per page)
- **Backlinks**: `serpstat_backlinks_summary` (5 credits), `serpstat_backlinks_top_anchors` (quick), `serpstat_backlinks_anchors`, `serpstat_backlinks_active`, `serpstat_backlinks_lost`, `serpstat_backlinks_ref_domains`, `serpstat_backlinks_top_pages`, `serpstat_backlinks_intersect` (link gap), `serpstat_backlinks_outlinks`, `serpstat_backlinks_out_domains`, `serpstat_backlinks_threats` (disavow)
- **Projects/limits**: `serpstat_projects_list`, `serpstat_credits_stats`, `serpstat_credits_audit_stats`
- **Rank Tracker** (free): `serpstat_rt_projects` → `serpstat_rt_project_regions` → `serpstat_rt_project_status` → `serpstat_rt_keyword_serp_history` / `serpstat_rt_url_serp_history` (use pageSize 20-50)
- **Site Audit** (free to read): `serpstat_site_audit_list` → `serpstat_site_audit_basic_info` / `serpstat_site_audit_report` → drill-down `serpstat_site_audit_error_elements` → `serpstat_site_audit_sub_elements`
- **One-Page Audit** (free to read): `serpstat_page_audit_pages_list` → `serpstat_page_audit_results` / `serpstat_page_audit_report_by_categories` → `serpstat_page_audit_error_rows`

## Proven workflows

### Domain SEO report (~100-200 credits)
1. `serpstat_credits_stats` → 2. `serpstat_domain_regions_count` → 3. `serpstat_domains_info` → 4. `serpstat_backlinks_summary` → 5. `serpstat_domain_keywords` (size 50-100, withIntents for g_us/g_ua) → 6. `serpstat_backlinks_top_anchors`. Summarize: visibility/traffic dynamics, position distribution, anchor profile, alerts.

### Quick wins (~300-500 credits)
1. `serpstat_domain_keywords` with `filters: {position_from: 4, position_to: 20, region_queries_count_from: 100}`, sort by traffic → 2. `serpstat_keywords_info` on the best candidates (exact volume/difficulty) → 3. for top pages `serpstat_url_missing_keywords` (sort weight desc). Output: prioritized list — keyword, current position, volume, page, concrete action (title/paragraph/internal link).

### Competitor gap (~200-400 credits)
1. `serpstat_competitors` → pick 1-2 relevant → 2. `serpstat_domain_uniq_keywords` (their keywords, minusDomain = user's domain, filter region_queries_count_from 100+) → 3. cluster by topic, map to existing/missing pages.

### Link gap
1. `serpstat_backlinks_summary` for user + competitors → 2. `serpstat_backlinks_intersect` (query = user domain, intersect = competitors) → domains linking to competitors but not to the user = outreach targets, sorted by domain_rank.

### Content plan for a keyword
1. `serpstat_keywords` (seed) → 2. `serpstat_related_keywords` → 3. `serpstat_keyword_suggestions` → 4. `serpstat_keywords_info` (bulk exact metrics) → cluster by intent (informational/commercial/transactional), propose page format per cluster.

### Toxic links audit
`serpstat_backlinks_summary` (referring_malicious_domains) → `serpstat_backlinks_threats` → group by threat_type, prepare disavow list.

## Interpreting fields

- `visible` — visibility index (share of top positions weighted by volume); compare only within the same `se`
- `difficulty` — 0-100, keyword difficulty; `concurrency` — 1-100, ads competition
- `region_queries_count` — monthly search volume (exact), `_wide` — broad match
- `weight` in missing keywords / related — how many competitors rank; higher = stronger signal
- `sersptat_domain_rank` (SDR, typo is Serpstat's own) — domain authority 0-100
- `types` — SERP features (snippets, video, also_asks...); presence changes CTR expectations
