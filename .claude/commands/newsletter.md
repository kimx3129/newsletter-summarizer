---
description: Run the full daily tech/AI newsletter pipeline (collect -> dedup -> rank -> summarize -> fact-check -> deliver to Slack)
---

You are the orchestrator for the daily newsletter pipeline. Run these six subagents **in strict sequence**, each via the Agent tool, passing the previous agent's full output as the input/context for the next. Do not skip, reorder, or parallelize steps — each depends on the previous one's output. Do not do the agents' work yourself; delegate.

1. `collector-agent` — no input needed beyond "collect today's candidates per config/interests.md". Produces a raw candidate list.
2. `dedup-agent` — input: the raw candidate list from step 1. Produces a deduped list.
3. `ranking-agent` — input: the deduped list from step 2. Produces the top 5–10 ranked stories.
4. `summarizer-agent` — input: the ranked list from step 3. Produces summaries.
5. `fact-check-agent` — input: the summarized list from step 4. Produces verified/corrected/flagged summaries.
6. `delivery-agent` — input: the fact-checked list from step 5. Formats, sends to Slack, archives to `history/`.

## Rules

- If any stage returns an empty or clearly broken result (e.g. collector-agent finds nothing, or delivery-agent reports `SLACK_WEBHOOK_URL` is missing), **stop the pipeline** and report exactly what happened instead of pushing broken/empty data to the next stage.
- After delivery-agent finishes, give the user a short final report: how many stories were delivered, how many flagged unverified, and the archive file path. Do not paste the entire digest content again if it was already sent to Slack — the user will see it there.
- If run without Slack configured (`SLACK_WEBHOOK_URL` unset), still run stages 1–5 so the user can see what *would* have been sent, then clearly say delivery was skipped and why.
