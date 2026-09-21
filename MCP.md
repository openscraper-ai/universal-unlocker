# OpenScraper MCP — scrape from Claude Code

Connect OpenScraper's scraping infrastructure to **Claude Code** (or any MCP
client). Ask Claude to scrape a site; it probes the target, tells you what
protects it and what it'll cost, then either hands you runnable code or runs the
job for you — all billed to your own API key.

Landing page: https://openscraper.ai/mcp

## 1. Get your API key
From your dashboard at https://openscraper.ai/dashboard — it starts with
`sk_live_`. Export it so the MCP can authenticate as you:
```bash
export OPENSCRAPER_API_KEY=sk_live_...
```
Put that line in your shell profile (`~/.zshrc` / `~/.bashrc`) so it's set
before Claude Code starts — the plugin reads it when it connects the server.

## 2. Install the plugin (recommended)
This installs the hosted MCP connection **and** the guided `vibe-scraper` skill
in one step, and keeps them updated:
```bash
claude plugin marketplace add openscraper-ai/universal-unlocker
claude plugin install openscraper@openscraper-ai
```
(Or interactively inside Claude Code: `/plugin marketplace add
openscraper-ai/universal-unlocker` then `/plugin install openscraper@openscraper-ai`.)

The server is hosted by us — you don't run anything. Your key is sent as a Bearer
header and used only to authenticate and bill your own account.

### Staying up to date
Claude Code **auto-updates** installed plugins in the background, so tool and
skill changes reach you without reinstalling. To pull the latest immediately:
```bash
claude plugin marketplace update openscraper-ai
```
The scraping *rules and tool behavior* live on our hosted server, so most updates
reach you automatically on your next session — the plugin update only refreshes
the local skill and the server list.

## Alternative: connect the MCP manually (other clients, no plugin)
If you're not using Claude Code's plugin system, add the server directly:
```bash
claude mcp add --transport http openscraper https://mcp.openscraper.ai/mcp \
  -H "Authorization: Bearer sk_live_..."
```
The tools ship with built-in guidance (the server sends its own instructions on
connect), so they work well even without the skill. To also get the guided
interview skill, copy it in:
```bash
mkdir -p ~/.claude/skills/vibe-scraper
curl -sL https://raw.githubusercontent.com/openscraper-ai/universal-unlocker/main/skills/vibe-scraper/SKILL.md \
  -o ~/.claude/skills/vibe-scraper/SKILL.md
```
(With the manual route you maintain updates yourself — the plugin route is why we
recommend it.)

## Then just ask
> "probe leboncoin.fr and scrape the car listings"

## Tools
| Tool | What it does |
|---|---|
| `probe_site(url)` | Detect protection, which engine gets through, volume + proxy recommendation |
| `list_modules()` | Available scrapers (dedicated managed modules) |
| `estimate_cost(module, params)` | Upper-bound price before you launch |
| `preview_sample(module, params, n)` | A small real sample of the data |
| `warm_session(url, proxy)` | Open a URL in our universal browser and get a replayable session (cookies + headers) to replay cheaply with curl_cffi — for open/JS-gated sites at scale |
| `run_scrape(module, params)` | Launch a full managed run (async) |
| `get_run` / `list_runs` | Track runs, fetch results |
| `cancel_run(id)` | Stop a run |
| `export_results(id)` | Get the full result set |
| `generate_client_code(module, params, lang)` | Runnable code to scrape from your own machine (curl_cffi + selectolax) |
| `get_balance()` | Your credit balance |

Everything is scoped to your key: you only see your own runs, and an empty
balance scrapes nothing.
