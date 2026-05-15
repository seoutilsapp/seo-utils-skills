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

## Rule 5: Visualizing GMB Rank Tracker Grids

When the user asks to "draw", "render", "show on a map", or "visualize" GMB rank tracker grids or rankings, produce an HTML artifact using **Leaflet 1.9.4 from the unpkg CDN** that matches the in-app heatmap style. Do NOT use Google Maps (needs an API key, won't render in artifacts) or default Leaflet pins (wrong style — the app uses small colored circles).

**Data query** — use `query_database` to fetch grid points:
- Grid layout only (no rankings): `SELECT lat, lng FROM google_business_rank_tracker_markers WHERE google_business_rank_tracker_report_id = ? AND disabled = 0`
- Grid with rankings: join `google_business_rank_tracker_markers` with `google_business_rank_tracker_snapshot_items` on `(lat, lng)`, filtered by `place_id = report.place_id` and the chosen `snapshot_id`. For "best rank across all keywords" per grid point use `MIN(rank)` grouped by `(lat, lng)`. See the `google_business_rank_tracker_snapshot_items` schema annotation for the canonical avg_rank / SoLV math — do not invent your own.

**Rendering recipe** — match the desktop heatmap (`frontend/src/views/GoogleBusinessRankTracker/GoogleBusinessRankTrackerReportHeatmap.vue`):

1. Load Leaflet from CDN: `https://unpkg.com/leaflet@1.9.4/dist/leaflet.css` + `https://unpkg.com/leaflet@1.9.4/dist/leaflet.js`.
2. Init the map read-only (all interaction disabled for a clean dashboard look):
   ```js
   L.map(el, {
     dragging:false, zoomControl:false, scrollWheelZoom:false, doubleClickZoom:false,
     touchZoom:false, boxZoom:false, keyboard:false, attributionControl:false
   })
   ```
   Skip the tile layer entirely for a transparent dot-only heatmap. Add `L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png')` only if the user explicitly asks for streets — and re-enable `dragging`/`zoomControl` if they want it interactive.
3. One marker per grid point with `L.divIcon` containing a 10×10 px colored circle:
   ```js
   L.marker([lat, lng], {
     icon: L.divIcon({
       className: 'gmb-card-marker',
       html: `<div style="background:${color};width:10px;height:10px;border-radius:9999px;border:1px solid rgba(0,0,0,0.15);"></div>`,
       iconSize: [10, 10],
       iconAnchor: [5, 5],
     }),
     interactive: false,
   }).addTo(map)
   ```
4. Color by rank — exact thresholds, do not improvise:
   - rank ≤ 3 → `#22c55e` (green)
   - rank 4–10 → `#eab308` (yellow)
   - rank > 10 OR no rank → `#ef4444` (red)
5. Auto-fit bounds: `map.fitBounds(bounds, {padding:[12,12], maxZoom:19})` — use padding `[6,6]` when there are ≥121 markers.

If the user wants a richer view (hover popups with rank/keyword, side-by-side per-keyword maps, streets), extend this recipe — but keep the 10-px dot size, the color thresholds, and the Leaflet 1.9.4 CDN URL the same so the artifact stays visually consistent with the desktop heatmap.
