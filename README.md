# SEO Utils MCP Guide Skill

Helps AI assistants use the [SEO Utils](https://seoutils.app) MCP server correctly — choosing the right tool (local database vs API) for SEO data queries.

## What does it do?

Without this skill, AI assistants sometimes call the wrong tool — for example, using `get_organic_keywords` (API) when you ask about your rank tracker reports (local data).

With the skill installed, the AI knows:
- "Show me my rank tracker report" → queries `organic_rank_tracker_*` tables locally
- "What keywords does competitor.com rank for?" → calls `get_organic_keywords` API
- "Find keyword cannibalization" → queries `search_console_query_pages` locally

## Installation

### Claude Desktop / Cowork / ChatGPT

1. [Download the latest ZIP](https://github.com/seoutilsapp/seo-utils-skills/archive/refs/heads/main.zip)
2. Go to **Customize → Skills**
3. Click **"+"** → **"Upload a skill"**
4. Upload the downloaded ZIP file
5. Toggle the skill on

### Claude Code

```bash
mkdir -p ~/.claude/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md \
  -o ~/.claude/skills/seo-utils-mcp-guide/SKILL.md
```

### Google Antigravity

```bash
mkdir -p ~/.gemini/antigravity/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md \
  -o ~/.gemini/antigravity/skills/seo-utils-mcp-guide/SKILL.md
```

### OpenClaw

```bash
mkdir -p ~/.openclaw/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md \
  -o ~/.openclaw/skills/seo-utils-mcp-guide/SKILL.md
```

### Perplexity

Download [Skill.md](https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md) and upload in the **My Skills** tab.

## Requirements

- [SEO Utils](https://seoutils.app) desktop app with MCP server enabled
- [MCP Access](https://app.seoutils.app/mcp/purchase) (one-time purchase)

## Links

- [MCP Setup Guide](https://help.seoutils.app/guide/mcp-server)
- [SEO Utils Website](https://seoutils.app)
