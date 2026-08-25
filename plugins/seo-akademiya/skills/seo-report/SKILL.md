---
name: seo-report
description: Build a client-ready SEO report as a PDF via seo-akademiya MCP. Use when the user says "SEO звіт", "отчет по SEO", "звіт для клієнта", "monthly report", "сделай отчет в pdf", or asks to prepare/export SEO results for a period.
---

# SEO report → PDF

Answer in the user's language. A report is NOT an analysis dump — it answers "як справи і що далі" for a specific reader.

## Phase 0 — ask BEFORE collecting anything

Ask these in ONE message (skip what the user already stated):
1. **Проєкт** — resolve via `find_project`.
2. **Період** — default: last full calendar month.
3. **Порівняння** — previous period AND/OR same period last year (seasonal businesses need YoY; if unsure, do both).
4. **Читач** — клієнт (простіше, висновки й гроші) чи внутрішня команда (деталі, задачі)?
5. **Мова звіту**.
6. **Обсяг** — короткий (2-3 стор.: KPI + висновки + план) чи повний (+семантика, посилання, техстан)?
7. **Що болить** — чи є питання, на яке звіт мусить відповісти (напр. "чому впали ліди")?

Wait for answers. Then estimate Serpstat credits (~30 short / ~100 full) and proceed.

## Phase 1 — collect (per chosen scope)

Follow the `seo-analysis` methodology, but ONLY the sections chosen, with both comparison windows everywhere:
- **KPI core (always)**: GSC clicks/impressions/CTR/position totals + by query (brand vs non-brand) + by page; GA4 organic sessions, key events, revenue if e-commerce; deltas vs comparison period(s).
- **Wins & losses (always)**: top-5 growing / top-5 falling queries and pages with the reason class (positions / demand / CTR / AI Overview — see traffic-drop logic).
- **Full scope adds**: visibility & competitors dynamics (`serpstat_domains_info` batch), backlinks delta (`serpstat_backlinks_summary`, new/lost), tech snapshot (`serpstat_site_audit_basic_info` SDO if a project exists, `gsc_list_sitemaps`), page-2 reserve list.
- Exclude last 2 days of GSC (data lag). Use equal-length windows. Cache makes re-runs free — collect first, write after.

## Phase 2 — write the report

Structure (each section = numbers + one-line "що це означає"):
1. **Титул**: проєкт, період, порівняння, дата, автор ("SEO Akademiya").
2. **Executive summary** — 4-6 речень: динаміка, головна причина, головний наступний крок. Пиши для людини, яка прочитає ТІЛЬКИ це.
3. **KPI-таблиця**: метрика | період | порівняння | Δ% | коментар. Стрілки ↑↓→, зелений/червоний.
4. **Динаміка** — по тижнях/днях (CSS bar chart, без зовнішніх бібліотек).
5. **Що виросло / що впало** — з причинами, не просто списки.
6. (повний) Конкуренти, посилання, техстан, резерв стор. 11-20.
7. **План на наступний період** — 3-7 пунктів: дія → підстава → очікуваний ефект.
8. Футер: джерела даних (GA4, GSC, Serpstat), дата генерації, застереження про приблизність сторонніх оцінок.

Rules: округлюй (4 210, не 4210.37), кожна цифра має порівняння, жодного сирого JSON, негатив не ховати — пояснювати причину й дію.

## Phase 3 — deliver as PDF

1. Write a single self-contained HTML file (inline CSS, no external resources; A4 print styles: `@page { size: A4; margin: 18mm }`, page-break between sections, brand-neutral clean design, tables with borders, CSS bars for charts).
2. Convert — try in this order, use the first available:
   - `google-chrome --headless --print-to-pdf=report.pdf report.html` (or `chromium`)
   - `wkhtmltopdf report.html report.pdf`
   - `weasyprint report.html report.pdf`
   - `pandoc report.html -o report.pdf`
3. If NONE is installed: save the HTML, tell the user "відкрий у браузері → Ctrl+P → Зберегти як PDF" — the print CSS makes it identical. Offer to install a converter if the environment allows.
4. File name: `seo-report-<domain>-<period>.pdf` (e.g. `seo-report-prostotv.com-2026-08.pdf`). Tell the user the full path.

## Pitfalls

- Never ship a report where numbers in summary differ from tables — recheck totals.
- GSC totals by query ≠ totals by date (sampling) — use the `date` dimension totals as canonical KPI.
- If the user asks monthly recurring reports, suggest saving the chosen settings (period type, scope, language) so next time only "звіт за серпень" is needed.
