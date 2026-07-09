# newsletter-summarizer

Daily tech/AI news digest pipeline. Six subagents run in sequence via `/newsletter`, each doing one job, so the final Slack message only contains the handful of stories actually worth reading.

## Pipeline

`collector-agent` → `dedup-agent` → `ranking-agent` → `summarizer-agent` → `fact-check-agent` → `delivery-agent`

Agent definitions: `.claude/agents/*.md`. Orchestrator (slash command): `.claude/commands/newsletter.md`. Run the whole thing with `/newsletter`.

## Config

`config/interests.md` — topics (priority order), target story count (5–10/day), recency window, ranking criteria. Edit this file to change what the digest prioritizes; no code changes needed.

## Delivery

Slack Incoming Webhook. Requires the `SLACK_WEBHOOK_URL` environment variable to be set wherever `/newsletter` runs — never hardcode it in any file in this repo.

## History

`history/YYYY-MM-DD.md` — archive of each day's delivered digest, written by `delivery-agent`. `dedup-agent` reads the last 3–5 days of these to avoid re-sending stories already covered.
