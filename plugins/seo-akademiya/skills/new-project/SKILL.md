---
name: new-project
description: Full onboarding audit for a NEW client project via seo-akademiya MCP. Use when the user says "новый проект", "новий проєкт", "взяли клиента", "стартовый аудит", "onboard", or asks for a first full look at a site they just started working on.
---

# New project onboarding audit

Answer in the user's language. Goal: one structured starting report + a work plan.

## Workflow (≈40-60 credits total)

1. **Resolve the project**: `find_project("<name>")`. If GA4 or GSC is missing from the result, tell the user which access is not granted yet — that is a finding, not a blocker.
2. **Pick the Serpstat database**: `serpstat_domain_regions_count` (~5 credits). Use the db with most keywords. Record it — every later serpstat_* call needs this `se`.
3. **Market position** (`serpstat_domains_info`, ~5 credits): visibility, keywords, traffic + dynamics. Negative `keywords_dynamic` on a new project = inherited decline; say so.
4. **Competitors** (`serpstat_competitors`, size 10): store top-5 relevant (same business type, not marketplaces/wiki) for later gap analysis.
5. **Organic reality — GSC** (`gsc_search_analytics`):
   - by `query`, rowLimit 25: separate BRAND vs NON-BRAND yourself (brand = contains company/domain name). Brand share > 60% of clicks = weak SEO, big upside.
   - by `page`, rowLimit 15: which pages actually earn clicks; note pages with impressions ≥ 500 and position 11-20 — the "page-2 reserve", fastest wins.
   - by `date` for 12+ months if available: seasonality shape.
6. **Traffic & conversions — GA4** (`ga4_run_report` channels + `ga4_key_events`): channel mix, whether conversions are configured at all. No key events = flag it first — without goals SEO cannot be evaluated.
7. **Backlinks** (`serpstat_backlinks_summary`, 5 credits): SDR, referring domains, nofollow share. Compare SDR with the top-3 competitors' (one more `serpstat_domains_info` batch or `backlinks_summary` per competitor — ask before spending if > 3 competitors).
8. **Tech state**: `serpstat_projects_list` — if the domain already has a project with audits, read `serpstat_site_audit_list` + `serpstat_site_audit_basic_info` (free). If not, OFFER to create a project and run an audit (MUTATING + costs 1 credit/page) — never start it silently.

## Output format

Report with sections: Доступи | Ринок і конкуренти | Органіка (бренд/небренд, резерв стор. 11-20) | Канали й конверсії | Посилання | Техстан | **План на 30 днів** (3-5 пунктів, кожен прив'язаний до знайденого факту).

## Pitfalls

- `rowCount: 0` from GA4 is not an error — property may be young or wrong id; re-check via find_project.
- GSC `siteUnverifiedUser` permission = data will be empty; report it as missing access.
- Do not run expensive serpstat tools (`url_summary_traffic` = 1000 credits) in onboarding.
