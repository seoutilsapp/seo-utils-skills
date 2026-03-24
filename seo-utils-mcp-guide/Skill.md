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
- Log Analysis → `log_analysis_*` tables
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
| "keyword cannibalization" | `get_organic_keywords` | `query_database` on `search_console_query_pages` |
| "trending queries" or "GSC data" | `get_organic_keywords` | `query_gsc` on `search_console_queries` |
| "my backlink history" | `fetch_backlinks` | Could be either — ask if they mean tracked data or fresh API data |
| "optimization opportunities" | `get_organic_keywords` | `query_database` on `search_console_query_pages` + `search_console_query_page_mentions` |
| "weak pages" or "low CTR pages" | `get_organic_keywords` | `query_gsc` on `search_console_pages` or `search_console_queries` |
| "my automations" | N/A | `query_database` on `automations` table |
| "content brief" or "content outline" | N/A | `query_database` on `content_struct_layout_headings` + `content_struct_layout_metadata` |

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
