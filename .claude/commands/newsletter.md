---
description: Run the full daily tech/AI newsletter pipeline (collect -> dedup -> rank -> summarize -> fact-check -> deliver)
---

You are the orchestrator for the daily newsletter pipeline. Run these six subagents **in strict sequence**, each via the Agent tool, passing the previous agent's full output as the input/context for the next. Do not skip, reorder, or parallelize steps — each depends on the previous one's output. Do not do the agents' work yourself; delegate.

1. `collector-agent` — no input needed beyond "collect today's candidates per config/interests.md and config/sources.md". Produces a raw candidate list.
2. `dedup-agent` — input: the raw candidate list from step 1. Produces a deduped/grouped list.
3. `ranking-agent` — input: the deduped list from step 2. Produces the top 5–10 ranked stories.
4. `summarizer-agent` — input: the ranked list from step 3. Produces 3-line + "why it matters" summaries.
5. `fact-check-agent` — input: the summarized list from step 4. Produces verified/corrected/flagged summaries.
6. `delivery-agent` — input: the fact-checked list from step 5. Formats, sends to every channel enabled in `config/delivery.md`, archives to `history/`.

After delivery-agent finishes and has written the `history/<date>.md` archive file, if this is a git repository with a remote, commit that new archive file and push it (e.g. `git add history/ && git commit -m "Add <date> digest archive" && git push`). This keeps the dedup history in sync across runs — without it, a fresh clone (e.g. a cloud scheduled run) won't see what was already sent and may repeat stories. If the push fails (no remote, no write access, merge conflict), don't treat it as a pipeline failure — just note it in the final report.

## Rules

- If any stage returns an empty or clearly broken result (e.g. collector-agent finds nothing from any source), **stop the pipeline** and report exactly what happened instead of pushing broken/empty data to the next stage.
- Delivery is per-channel, not all-or-nothing: if some channels in `config/delivery.md` are enabled but missing credentials, or a send to one channel fails, that does not mean the pipeline failed — delivery-agent reports per-channel status, and you should relay that as-is rather than treating it as an overall failure.
- After delivery-agent finishes, give the user a short final report: per-channel delivery status, how many stories were delivered, how many flagged unverified, and the archive file path. Do not paste the entire digest content again if it was already sent somewhere the user will see it.
- If no configured channel actually has credentials set, still run stages 1–5 so the user can see what *would* have been sent, then clearly say delivery was skipped entirely and why.
