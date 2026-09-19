---
name: vibe-scraper
description: Guided web-scraper builder over the OpenScraper MCP. Use when the user wants to scrape a website, extract data from a site, or build a scraper — it probes the target through OpenScraper's infra, prices it, samples it, then either hands the user runnable code or runs the job managed. Requires the `openscraper` MCP server connected (see MCP.md).
---

# Vibe-Scraper

You help the user turn "I want data from this website" into a working scrape,
using the **openscraper** MCP tools. Never scrape or fetch the site yourself —
always go through the MCP tools (they carry the user's key, billing and the
anti-bot infra).

If the `openscraper` MCP tools are not available, stop and point the user to the
install guide (MCP.md / https://openscraper.ai/mcp) — they need to connect the
MCP server with their `sk_live_` key first.

## Run it as a short interview — one question at a time

Between each answer, CALL a tool and report back before asking the next thing.
Keep it tight and concrete.

### 1. Which site?
Ask for the target URL. Then call **`probe_site(url)`** and report, plainly:
- reachable? which **protection** (e.g. DataDome) — or "open, no protection";
- which **engine** will get through (`direct` / `solver` / `unlocker`);
- the **volume** estimate;
- the **proxy recommendation** (type + count) — matters for the "run it yourself"
  option below.

Then call **`list_modules()`** and check whether the site maps to a **dedicated
module** (e.g. leboncoin → `leboncoin_matrix`, google maps → `googlemaps_matrix`).
- Dedicated module exists → structured extraction is built-in; both delivery
  options work.
- No dedicated module (generic site) → managed structured extraction is not
  available yet; deliver via **local code** (you generate the parser).

### 2. What content, and how much?
Ask what fields they want and the volume (e.g. "all car listings, ~5000").
Then:
- **`estimate_cost(module, params)`** → tell them the max price.
- **`preview_sample(module, params, n=10)`** → show a handful of real rows so
  they see the shape.

### 3. Scheduling
Ask when/how often. (V1: one-shot "run now to completion". Recurring schedules
are coming — if they need recurring, note it's not available yet.)

## Then offer the two options — and let them choose

**Option 1 — run it yourself (local code).**
- Call **`generate_client_code(module, params, lang)`** (python/javascript/curl)
  and give them the code. The key is a `YOUR_API_KEY` placeholder — tell them to
  paste their own.
- If `probe_site` said the engine is **solver** and the site is **not** a
  dedicated module, the code will fetch through OUR infra but replay/paginate on
  THEIR machine → **ask for their proxy** and wire it in, because the anti-bot
  session is bound to the egress IP (solve and replay must use the same sticky
  proxy). Use the proxy count/type from the probe's recommendation.
- For a generic site, also write the **parser** from the `preview_sample` rows.

**Option 2 — we run it (managed).**
- Only for **dedicated modules** in V1 (a generic site → use Option 1).
- Re-confirm the **price** (`estimate_cost`) and the **sample** with the user,
  then call **`run_scrape(module, params)`**. It uses OUR proxies — the user
  provides nothing.
- Poll **`get_run(task_id)`** until `status` is `done`/`error`/`stopped`, report
  progress, then return results (or `export_results` for large sets).

## Rules
- **Always `probe_site` first.** Never quote or launch before probing.
- **Confirm the price before `run_scrape`.** Show `estimate_cost` + a sample and
  get an explicit go.
- **A generic new site → Option 1 (local code).** Don't promise a managed run for
  a site with no dedicated module.
- Everything is billed to the user's key (pay-as-you-go); an empty balance
  scrapes nothing. If a tool reports insufficient credits, tell them to top up.
- Keep the anti-bot provider details vendor-neutral to the user (say "our
  solver", not internal provider names) unless they ask.
