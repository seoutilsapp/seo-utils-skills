# SEO Utils MCP Guide Skill

Helps AI assistants use the [SEO Utils](https://seoutils.app) MCP server correctly — choosing the right tool (local database vs paid lookup vs write) for SEO data queries.

## What does it do?

Without this skill, AI assistants sometimes call the wrong tool — for example, looking up a domain's keywords through DataForSEO (which spends credits) when you ask about your own rank tracker reports (local data).

With the skill installed, the AI knows:
- "Show me my rank tracker report" → queries `organic_rank_tracker_*` tables locally
- "What keywords does competitor.com rank for?" → runs the `get_organic_keywords` lookup action
- "Find keyword cannibalization" → queries `search_console_query_pages` locally
- "Delete my GMB test reports" → finds the reports, confirms them with you, then runs `delete_gmb_rank_tracker_reports`

## Installation

### Claude (Claude Code, Claude Desktop, claude.ai, Cowork): install the plugin

**Claude Code:**

```
/plugin marketplace add seoutilsapp/seo-utils-skills
/plugin install seo-utils@seo-utils
```

To get skill updates automatically, open `/plugin` → **Marketplaces** → **seo-utils** → **Enable auto-update** (Claude Code leaves it off for marketplaces you add). Or update by hand with `claude plugin update seo-utils@seo-utils`.

**Claude Desktop or claude.ai:** open **Customize → Plugins**, add the marketplace `seoutilsapp/seo-utils-skills`, then install **SEO Utils**. A plugin you install there is also available in Claude Code. To get skill updates, turn on **Sync automatically** for the marketplace, or select **Check for updates**.

**Installed the skill before, with the `curl` command or an upload?** Remove that copy once the plugin is installed, so Claude doesn't load two versions: `rm -rf ~/.claude/skills/seo-utils-mcp-guide` for Claude Code, and delete it under **Customize → Skills** in Claude.

The plugin contains the skill only. Connect the MCP server itself from the SEO Utils app (**Settings → MCP Server**).

### Upload the skill file instead

Use this for assistants without plugins, or if you prefer to manage the file yourself. It doesn't update automatically.

**Claude Desktop / Cowork / ChatGPT:** these take a ZIP holding a `seo-utils-mcp-guide` folder with `SKILL.md` inside (the repository's own ZIP doesn't have that shape).

1. Get the ZIP: in SEO Utils, open **Settings → MCP Server**, choose your app and select **Save skill ZIP**. Or build it: create a folder named `seo-utils-mcp-guide`, save [SKILL.md](https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/plugins/seo-utils/skills/seo-utils-mcp-guide/SKILL.md) in it, and zip the folder.
2. Claude: go to **Customize → Skills → + → Create skill → Upload a skill**. ChatGPT: go to **Plugins → Skills → Create → Upload from your computer** (Business, Enterprise or Edu plans).
3. Upload the ZIP and turn the skill on.

**Claude Code:**

```bash
mkdir -p ~/.claude/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md \
  -o ~/.claude/skills/seo-utils-mcp-guide/SKILL.md
```

**Google Antigravity:**

```bash
mkdir -p ~/.gemini/antigravity/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md \
  -o ~/.gemini/antigravity/skills/seo-utils-mcp-guide/SKILL.md
```

**OpenClaw:**

```bash
mkdir -p ~/.openclaw/skills/seo-utils-mcp-guide
curl -sL https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md \
  -o ~/.openclaw/skills/seo-utils-mcp-guide/SKILL.md
```

**Perplexity:** download [Skill.md](https://raw.githubusercontent.com/seoutilsapp/seo-utils-skills/main/Skill.md) and upload it in the **My Skills** tab.

## Requirements

- [SEO Utils](https://seoutils.app) desktop app with MCP server enabled
- [MCP Access](https://app.seoutils.app/mcp/purchase) (one-time purchase)

## Updating the skill (maintainers)

- Edit `plugins/seo-utils/skills/seo-utils-mcp-guide/SKILL.md`, then copy it to `Skill.md`. The root `Skill.md` serves the download links above and older install instructions; a check fails when the two differ.
- The plugin lives in `plugins/seo-utils/`, apart from the root `Skill.md`: Claude Desktop and claude.ai reject a plugin that holds two copies of one skill.
- Don't add a `version` to `plugins/seo-utils/.claude-plugin/plugin.json`. Without one, every commit on `main` is a new version, so installed plugins update. A pinned version keeps everyone on the old copy until someone changes it.

## Links

- [MCP Setup Guide](https://help.seoutils.app/guide/mcp-server)
- [SEO Utils Website](https://seoutils.app)
