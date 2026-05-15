---
name: seo-utils-mcp-guide
description: Background knowledge for using SEO Utils MCP tools correctly. Helps choose the right tool (local database vs API) for SEO data queries.
---

# SEO Utils MCP Tool Selection Guide

When the user asks about SEO data, use this guide to pick the correct tool.

## Rule 1: Local Data vs API Data

The user has TWO types of SEO data:

**Local database** (synced/tracked over time) — use `query_database` or `query_gsc`:
- Organic Rank Tracker reports → `organic_rank_tracker_*` tables
- Google Business Rank Tracker → `google_business_rank_tracker_*` tables
- Google Search Console data → `search_console_*` tables (use `query_gsc`)
- LLM Rank Tracker → `llm_rank_tracker_*` tables
- Content Struct outlines → `content_struct*` tables
- NLP Analysis → `nlp_analysis_*` tables
- NAP Finder → `nap_finder_*` tables
- Saved Keywords → `saved_keywords_*` tables
- Automations → `automations`, `automation_*` tables
- Indexing status → `site_urls`, `index_now_urls` tables
- Log Analysis → `log_analysis_reports`, `log_file_analysis_*` tables, per-report `log_sources` (auto-import config), per-hit `log_file_analysis_ai_requests` (AI answer/assistant bots, 90d retention), and the GLOBAL `bot_categories` lookup (bot → bucket: ai_answer / ai_assistant / ai_training / search / social / seo_tool / monitoring / other). For upgraded reports (`log_analysis_reports.bucket_schema_version >= 1`), `log_file_analysis_page_activities` has denormalized `ai_answer_hits` / `ai_assistant_hits` / `ai_training_hits` / `search_hits` / `other_bot_hits` columns; for pre-upgrade reports those are 0 and you must parse `bot_hits` JSON + JOIN `bot_categories`.
- SEO Tests → `tests`, `test_*` tables
- GA4 organic traffic + conversions → `ga4_landing_page_metrics` (daily, organic-search-only, per landing page; includes `key_events`, `revenue`, `engaged_sessions`, `bounce_rate`), `ga4_property_mappings`, `ga4_sync_statuses`
- LLM Rank Tracker → `llm_rank_tracker_reports`, `llm_rank_tracker_responses`, `llm_rank_tracker_positions` (brand/competitor mentions in ChatGPT / Claude / Gemini / Google AI Overview / Google AI Mode), `llm_rank_tracker_citations`, `llm_rank_tracker_competitors`
- NAP Finder → `nap_finder_reports`, `nap_finder_items`, `nap_finder_search_terms`
- SERP Clustering → `serp_clustering_reports`, `serp_clustering_keywords`, `serp_items` (group keywords by SERP overlap)
- Content / Topic clusters → `search_console_content_clusters` (page clusters via path filters or manual selection) and `topic_clusters` (GSC query clusters); metrics computed dynamically from `search_console_pages` / `search_console_queries`
- Keyword insights (pre-computed) → `keyword_insights` (`trending_up`, `trending_down`, `pogo_sticking`, `flickering`). Note: `key_events` insight is NOT stored — `get_keyword_insight_summary` computes it on-demand from GA4
- Google Business Reviews → `google_business_reviews`, `google_business_review_snapshots`
- People Also Ask scraping → `google_paa_reports`, `google_paa_keywords` (parent → sub-question tree), `google_paa_seed_keywords`
- NLP Analysis → `nlp_analysis_*` tables (entities + topics via TextRazor / other providers)
- Workspaces → `workspaces` (read `seo-utils://status` for the active workspace; use `set_workspace` to switch, `create_workspace`/`update_workspace` for CRUD)

**DataForSEO API** (on-demand, third-party estimates) — use action tools:
- `get_keyword_suggestions` / `get_bing_related_keywords` — keyword research
- `get_organic_keywords` — what keywords a domain ranks for (estimated)
- `get_traffic_summary` / `get_historical_rank` — traffic estimates
- `get_traffic_competitors` / `get_top_pages` — competitor/page analysis
- `fetch_backlinks` / `get_backlink_summary` / etc. — backlink data
- `bulk_traffic_analysis` / `bulk_backlink_analysis` — multi-domain comparison
- `check_keyword_metrics` — search volume, KD, CPC
- `fetch_serp_data` — live SERP results

## Rule 2: Common Mistakes to Avoid

| User says | WRONG tool | CORRECT approach |
|-----------|-----------|-----------------|
| "rank tracker report" or "my rankings" | `get_organic_keywords` | `query_database` on `organic_rank_tracker_*` tables |
| "add keywords to my rank tracker" or "start tracking X for example.com" | `add_keywords_to_list` (saved-keywords tool) or `query_database` (SQL is read-only, can't INSERT) | `add_organic_rank_tracker_keywords` — then ASK the user whether to `run_rank_tracker` as a follow-up (don't auto-rerun). Saved keyword lists are a separate feature |
| "keyword cannibalization" | `get_organic_keywords` | `query_database` on `search_console_query_pages` |
| "trending queries" or "GSC data" | `get_organic_keywords` | `query_gsc` on `search_console_queries` |
| "my backlink history" | `fetch_backlinks` | Could be either — ask if they mean tracked data or fresh API data |
| "optimization opportunities" | `get_organic_keywords` | `query_database` on `search_console_query_pages` + `search_console_query_page_mentions` |
| "weak pages" or "low CTR pages" | `get_organic_keywords` | `query_gsc` on `search_console_pages` or `search_console_queries` |
| "my automations" | N/A | `query_database` on `automations` table |
| "content brief" or "content outline" | N/A | `query_database` on `content_struct_layout_headings` + `content_struct_layout_metadata` |
| "which pages are AI bots fetching?" | `get_organic_keywords` | Check `log_analysis_reports.bucket_schema_version` first. If ≥1, `query_database` on `log_file_analysis_page_activities WHERE ai_answer_hits + ai_assistant_hits > 0`. If =0, parse `bot_hits` JSON + JOIN `bot_categories` |
| "what questions are AI engines asking about my site?" | N/A | `query_database` on `log_file_analysis_ai_requests` grouping by `extracted_query`. Low counts are NORMAL if traffic is mostly ChatGPT-User / Claude-User — those bots strip prompt data |
| "is GPTBot / ClaudeBot / Google-Extended violating my robots.txt?" | `query_database` | `get_robots_compliance` — SQL cannot evaluate robots.txt rules, the matcher is Go-side |
| "how is AI traffic trending?" | N/A | `query_database` on `log_file_analysis_page_activities` summing `ai_answer_hits + ai_assistant_hits` by date (upgraded reports only) |
| "which pages convert?" or "pages driving revenue/sign-ups" | `get_organic_keywords` / `get_top_pages` | `query_database` on `ga4_landing_page_metrics WHERE key_events > 0` (filter by `domain`). For revenue: `SUM(revenue)`. Note conversions are page-level, NOT keyword-level — never attribute a key event to a single keyword |
| "trending up / trending down keywords" | `get_organic_keywords` | `query_database` on `keyword_insights WHERE insight_type IN ('trending_up','trending_down')` filtered by `organic_rank_tracker_report_id`. For Key Events insight, use `get_keyword_insight_summary` (not stored in table) |
| "pogo-sticking" or "flickering" keywords | N/A | `query_database` on `keyword_insights WHERE insight_type IN ('pogo_sticking','flickering')`. These require **daily** tracking (`report.schedule = 1`) — return 0 rows for weekly/monthly reports |
| "how do I rank in ChatGPT / Claude / Gemini / AI Overview?" | `get_organic_keywords` | `query_database` on `llm_rank_tracker_positions WHERE position > 0 AND is_best_position = 1` filtered by `report_id`. `position = 0` means NOT mentioned, NOT NULL |
| "find my NAP citations" | N/A | `query_database` on `nap_finder_items` filtered by `nap_finder_report_id` |
| "cluster my queries" or "topic clusters" | N/A | `query_database` on `topic_clusters` + `topic_cluster_query_items` (JOIN `search_console_queries` for metrics). For page clusters, use `search_console_content_clusters` |
| "URL inspection" or "indexing status" | N/A | `query_database` on `site_urls` (filter by `domain`). For status changes over time, `indexing_movements`. For batch operations, `trigger_indexing_action` |
| "internal links" or "orphan pages" | N/A | `trigger_indexing_action` with `trigger_link_crawl` first, then `query_database` on `internal_links` (filter by `domain`). JOIN `site_urls` via `normalized_path` |
| "switch to workspace X" or "what workspace am I in?" | N/A | Read `seo-utils://status` for active workspace. Use `set_workspace` to switch. The status resource also returns `gsc_utc_offset_hours` — use it when extracting calendar dates from UTC-stored GSC datetimes |

## Rule 3: When to Use API Tools

Use DataForSEO API action tools ONLY when:
- User asks about a **domain they don't track** (competitor research)
- User explicitly asks for **fresh/live data** from DataForSEO
- User wants **keyword suggestions** or **keyword metrics** for new keywords
- User wants **backlink data** for any domain
- User wants to **compare multiple domains** (bulk analysis)

## Rule 4: Export and Send

- `send_email` — send any email with SMTP. Check `smtp_credentials` table first
- Automations — use `create_automation` for recurring workflows (triggered by events)
- For one-time exports, generate the content and use `send_email` to deliver it

## Rule 4.5: GA4 + Rankings Correlation

When the user asks "which keywords drive conversions" or "what's my best-converting topic":

1. Conversions in GA4 are **page-level**, not keyword-level. You can only correlate, never attribute. Always say this out loud when answering.
2. Find converting pages: `SELECT landing_page_path, SUM(key_events) AS conv, SUM(revenue) AS rev FROM ga4_landing_page_metrics WHERE domain = ? AND date >= ? GROUP BY landing_page_path ORDER BY conv DESC`.
3. Find keywords ranking on those pages: JOIN `organic_rank_tracker_positions` → `organic_rank_tracker_keywords` → `organic_rank_tracker_snapshots`, matching the page URL to `landing_page_path`. Use `MIN(position)` for best rank.
4. For Key Events at the keyword level inside a rank tracker report, use `get_keyword_insight_summary` (computed on-demand) — this is the only place the GA4 ↔ rank tracker join is materialized.
5. GA4 metrics are organic-search only (filtered by `sessionDefaultChannelGroup = 'Organic Search'`) — never present them as total site traffic.

## Rule 4.6: Workspace Awareness

Most data tables are workspace-scoped via `workspace_id`. Before answering questions like "show me all my rank tracker reports" or "what's my GSC data":

1. Read `seo-utils://status` to get the active workspace (`active_workspace.id`).
2. If a user references "another project" or "the other client", check `list_workspaces` and confirm before switching with `set_workspace`.
3. Switching workspaces affects every subsequent query — call it out in your response so the user isn't surprised.

## Rule 5: Visualizing GMB Rank Tracker Grids

When the user asks to "draw", "render", "show on a map", or "visualize" GMB rank tracker grids or rankings, produce an HTML artifact using **Leaflet 1.9.4 from the unpkg CDN** that matches the in-app heatmap style. Do NOT use Google Maps (needs an API key, won't render in artifacts) or default Leaflet pins (wrong style — the app uses small colored circles).

**Data query** — use `query_database` to fetch grid points:
- Grid layout only (no rankings): `SELECT lat, lng FROM google_business_rank_tracker_markers WHERE google_business_rank_tracker_report_id = ? AND enabled = 1`
- Grid with rankings: join `google_business_rank_tracker_markers` with `google_business_rank_tracker_snapshot_items` on `(lat, lng)`, filtered by `place_id = report.place_id` and the chosen `snapshot_id`. For "best rank across all keywords" per grid point use `MIN(rank)` grouped by `(lat, lng)`. See the `google_business_rank_tracker_snapshot_items` schema annotation for the canonical avg_rank / SoLV math — do not invent your own.

**Pick the right marker size — this is the most common mistake:**

| Context | Marker style | Mirrors |
|---|---|---|
| **Full-page / standalone artifact** (default when the user asks "draw a map", "show me the grid", "visualize my rankings") | 32 px circle with the **rank number inside** + 2 px white border | Google Maps view in `GoogleBusinessRankTrackerMap.vue` (`createMarkerContentWithRank`, `frontend/src/helpers/googleMaps.js:81-250`) |
| **Compact / dashboard thumbnail** (only when the user says "thumbnail", "card", "widget", "compact preview", or marker count is ≥225 and labels would overlap) | 10 px solid colored dot, no text | Heatmap card in `GoogleBusinessRankTrackerReportHeatmap.vue` |

**Rendering recipe — full-page (default):**

1. Load Leaflet from CDN: `https://unpkg.com/leaflet@1.9.4/dist/leaflet.css` + `https://unpkg.com/leaflet@1.9.4/dist/leaflet.js`.
2. Init the map **interactive** with an OpenStreetMap tile layer so the user can see streets/neighborhoods:
   ```js
   const map = L.map(el)
   L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
     attribution: '&copy; OpenStreetMap contributors', maxZoom: 19
   }).addTo(map)
   ```
3. One marker per grid point with a labeled `L.divIcon` (text = rank, or `21+` when no rank):
   ```js
   L.marker([lat, lng], {
     icon: L.divIcon({
       className: 'gmb-rank-marker',
       html: `<div style="display:flex;align-items:center;justify-content:center;
                          width:32px;height:32px;border-radius:9999px;
                          background:${color};color:#fff;font-weight:600;font-size:13px;
                          border:2px solid #fff;box-shadow:0 1px 3px rgba(0,0,0,0.35);">
                ${rank ?? '21+'}
              </div>`,
       iconSize: [32, 32],
       iconAnchor: [16, 16],
     }),
   }).addTo(map).bindTooltip(`Rank: ${rank ?? '21+'}`, {direction:'top'})
   ```
4. Auto-fit bounds: `map.fitBounds(bounds, {padding:[24,24], maxZoom:18})`.

**Rendering recipe — compact thumbnail (only when explicitly asked):**

- Same CDN load, but init map read-only and **omit the tile layer** for a transparent dot-only heatmap:
  ```js
  L.map(el, {dragging:false, zoomControl:false, scrollWheelZoom:false, doubleClickZoom:false,
             touchZoom:false, boxZoom:false, keyboard:false, attributionControl:false})
  ```
- Markers: `width:10px;height:10px;border-radius:9999px;border:1px solid rgba(0,0,0,0.15);` no text, `interactive:false`.
- Fit bounds with `padding:[12,12]` (use `[6,6]` when ≥121 markers).

**Color by rank — exact thresholds, do not improvise (both modes):**
- rank ≤ 3 → `#22c55e` (green)
- rank 4–10 → `#eab308` (yellow)
- rank > 10 (i.e. 11–20) OR no rank → `#ef4444` (red)

If the user wants extras (hover popups with rank/keyword/title, side-by-side per-keyword maps, dark mode), build on top of these recipes — but keep the marker dimensions, color thresholds, and Leaflet 1.9.4 CDN URL unchanged so the artifact stays visually consistent with the desktop app.
