# SEO Utils Skills

Custom skills for [SEO Utils](https://seoutils.app) that help AI assistants use the MCP server more effectively.

## Available Skills

| Skill | Description |
|-------|-------------|
| **seo-utils-mcp-guide** | Helps Claude choose the right MCP tool (local database vs API) for SEO data queries |

## Installation

### Claude Desktop / Cowork

1. [Download the latest ZIP](https://github.com/seoutilsapp/seo-utils-skills/archive/refs/heads/main.zip)
2. In Claude Desktop, go to **Customize → Skills**
3. Click **"+"** → **"Upload a skill"**
4. Upload the downloaded ZIP file
5. Toggle the skill on

### Claude Code

```bash
# Clone to your personal skills directory
git clone https://github.com/seoutilsapp/seo-utils-skills.git ~/.claude/skills/seo-utils-skills
```

Or copy just the skill you need:

```bash
mkdir -p ~/.claude/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/seo-utils-mcp-guide/Skill.md \
  -o ~/.claude/skills/seo-utils-mcp-guide/SKILL.md
```

## What does the MCP Guide skill do?

It teaches Claude when to use **local database queries** vs **DataForSEO API tools**. Without it, Claude sometimes calls the wrong tool — for example, using `get_organic_keywords` (API) when you ask about your rank tracker reports (local data).

With the skill installed, Claude knows:
- "Show me my rank tracker report" → queries `organic_rank_tracker_*` tables locally
- "What keywords does competitor.com rank for?" → calls `get_organic_keywords` API
- "Find keyword cannibalization" → queries `search_console_query_pages` locally

## Requirements

- [SEO Utils](https://seoutils.app) desktop app with MCP server enabled
- [MCP Access](https://app.seoutils.app/mcp/purchase) (one-time purchase)

## Links

- [MCP Setup Guide](https://help.seoutils.app/guide/mcp-server)
- [SEO Utils Website](https://seoutils.app)
