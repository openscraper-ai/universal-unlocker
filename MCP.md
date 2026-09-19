# OpenScraper MCP — scrape from Claude Code

Connect OpenScraper's scraping infrastructure to **Claude Code** (or any MCP
client) with one command. Ask Claude to scrape a site; it probes the target,
tells you what protects it and what it'll cost, then either hands you runnable
code or runs the job for you — all billed to your own API key.

Landing page: https://openscraper.ai/mcp

## 1. Get your API key
From your dashboard at https://openscraper.ai/dashboard — it starts with
`sk_live_`.

## 2. Connect the MCP server
```bash
claude mcp add --transport http openscraper https://mcp.openscraper.ai/mcp \
  -H "Authorization: Bearer sk_live_..."
```
The server is hosted by us — you don't run anything. Your key is sent as a
Bearer header and used only to authenticate and bill your own account.

## 3. (Optional) Install the vibe-scraper skill
The [`vibe-scraper`](skills/vibe-scraper/SKILL.md) skill scripts the guided
interview (probe → price → sample → code-or-run). Copy it into your Claude Code
skills directory:
```bash
mkdir -p ~/.claude/skills/vibe-scraper
curl -sL https://raw.githubusercontent.com/openscraper-ai/universal-unlocker/main/skills/vibe-scraper/SKILL.md \
  -o ~/.claude/skills/vibe-scraper/SKILL.md
```
Or just ask Claude directly — the MCP tools work without the skill; the skill
only makes the flow more guided.

## Then just ask
> "probe leboncoin.fr and scrape the car listings"

## Tools
| Tool | What it does |
|---|---|
| `probe_site(url)` | Detect protection, which engine gets through, volume + proxy recommendation |
| `estimate_cost(module, params)` | Upper-bound price before you launch |
| `preview_sample(module, params, n)` | A small real sample of the data |
| `run_scrape(module, params)` | Launch a full managed run (async) |
| `get_run` / `list_runs` | Track runs, fetch results |
| `cancel_run(id)` | Stop a run |
| `export_results(id)` | Get the full result set |
| `generate_client_code(module, params, lang)` | Runnable code to scrape from your own machine |
| `get_balance()` | Your credit balance |
| `list_modules()` | Available scrapers |

Everything is scoped to your key: you only see your own runs, and an empty
balance scrapes nothing.
