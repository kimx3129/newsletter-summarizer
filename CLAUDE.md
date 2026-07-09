# newsletter-summarizer

Daily tech/AI news digest pipeline. Six subagents run in sequence via `/newsletter`, each doing one job, so the final digest only contains the handful of stories actually worth reading.

## Pipeline

`collector-agent` → `dedup-agent` → `ranking-agent` → `summarizer-agent` → `fact-check-agent` → `delivery-agent`

Agent definitions: `.claude/agents/*.md`. Orchestrator (slash command): `.claude/commands/newsletter.md`. Run the whole thing with `/newsletter`.

## Config

- `config/interests.md` — topics (priority order), target story count (5–10/day), recency window, ranking criteria.
- `config/sources.md` — the fixed source list collector-agent pulls from: Hacker News, GitHub Trending, arXiv, Reddit, company/lab blogs, general tech RSS.
- `config/delivery.md` — which delivery channels are enabled (Slack/Discord/Gmail/Notion) and which env var each one needs.

Edit these directly to change behavior; no code changes needed.

## Delivery

Multi-channel: Slack, Discord, Gmail, Notion — see `config/delivery.md` for which are enabled and their required env vars (`SLACK_WEBHOOK_URL`, `DISCORD_WEBHOOK_URL`, `GMAIL_ADDRESS`/`GMAIL_APP_PASSWORD`/`GMAIL_TO`, `NOTION_API_KEY`/`NOTION_DATABASE_ID`). These must be set wherever `/newsletter` runs — never hardcode any of them in a file in this repo. A channel enabled without credentials set is skipped, not a pipeline failure.

## History

`history/YYYY-MM-DD.md` — archive of each day's delivered digest, written by `delivery-agent`. `dedup-agent` reads the last 3–5 days of these to avoid re-sending stories already covered.
