---
name: traffic-drop
description: Diagnose organic/overall traffic drops via seo-akademiya MCP. Use when the user says "упал трафик", "падіння трафіку", "просели позиции", "trafic drop", "почему меньше переходов", or asks why clicks/sessions declined.
---

# Traffic drop diagnosis

Answer in the user's language. Goal: name the CAUSE with evidence, not just describe the drop.

## Step 0 — frame it

Ask (or infer) the drop period. Build two windows of EQUAL length: current vs previous (and same period last year if seasonality is possible). Resolve ids via `find_project`.

## Step 1 — which channel (GA4)

`ga4_run_report` by `sessionDefaultChannelGroup` for both windows. If only paid/social fell — stop, it is not an SEO problem; say so. If Organic fell → continue.

## Step 2 — when exactly (GSC by date)

`gsc_search_analytics` by `date` across both windows. Sharp cliff on a specific date → suspect: Google update, site release, tracking loss. Slow slide → demand or competitors.

## Step 3 — the four-way split (core of the skill)

Compare the two windows by `query` (rowLimit 250) and compute per query: Δclicks, Δimpressions, Δposition, ΔCTR. Then classify each top loser:

| Pattern | Diagnosis |
|---|---|
| position worse by >2 | **Rankings lost** → check who took the spot: `serpstat_keyword_full_top` for 2-3 such queries |
| position stable, impressions down | **Demand fell** (season/news) → verify with `serpstat_keywords_info` volume; compare year-over-year in GSC |
| position & impressions stable, CTR down | **Snippet/SERP change** → new SERP features stealing clicks; check `serpstat_domain_keywords` filters for SERP features |
| impressions → 0 | **Deindexed/technical** → `gsc_inspect_url` on the URL |

Also split brand vs non-brand: brand fell = business/offline problem, not SEO.

## Step 4 — what changed on our side

- `ga4_change_history` — did someone change property/streams/key events around the cliff date (explains "analytics drop" without real drop). It needs the ACCOUNT id, not property id: take it from `ga4_property_info` → `property.account` ("accounts/305563768" → use "305563768").
- `gsc_list_sitemaps` — sitemap errors, last downloaded.
- `gsc_inspect_url` on 2-3 top losing pages — index state, canonical flips.
- `serpstat_site_audit_list` if project exists — did SDO score fall between audits.
- If Rank Tracker has the project: `serpstat_rt_url_serp_history` for exact position timeline.

## Output format

**Діагноз** (one sentence: what fell and why) → **Докази** (numbers per step) → **Що робити** (actions matched to the diagnosis) → **Що виключено** (checked and clean).

## Pitfalls

- Always compare equal-length windows; GSC data lags ~2 days — exclude the last 2 days.
- A "drop" after ~16 months may be GSC data retention limit, not reality.
- prostotv-style case: positions stable + impressions down = demand, do not promise "SEO fixes" for it.
