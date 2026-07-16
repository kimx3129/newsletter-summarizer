---
name: debug-slack-delivery
description: Diagnoses and fixes Slack delivery failures in the newsletter-summarizer pipeline — use when the daily digest doesn't show up in Slack, when delivery-agent reports (or is suspected to have silently failed) a Slack send, or when asked to debug/troubleshoot Slack webhook/delivery issues in this project.
---

You are debugging why a Slack message from this pipeline didn't arrive. Don't guess and don't trust a single subagent's self-report of success — a prior "HTTP 200 / ok" report earlier in this project's history described a message the user never actually saw, and the real cause turned out to be an entirely different layer (a cloud routine that had auto-disabled before delivery-agent ever ran). Isolate the failure to a specific layer before touching any code.

## Failure layers, in the order to check them

Slack messages from this project can fail for reasons that have nothing to do with delivery-agent's code. Check top-down; stop at the first layer that reproduces the failure, fix it, then re-verify end to end.

### 1. Is the webhook itself alive?

Test it directly, bypassing the whole pipeline:

```bash
curl -sS -w "\nHTTP_STATUS:%{http_code}\n" -X POST -H 'Content-Type: application/json' \
  --data '{"text":"diagnostic test"}' \
  "$SLACK_WEBHOOK_URL"
```

- `ok` / `200` → webhook and channel are fine. The problem is upstream — go to step 2.
- `404`/`no_service`/`invalid_token` → the webhook was revoked or the URL is wrong. It must be regenerated in Slack (Apps → Incoming Webhooks) — this is not fixable from code; tell the user.
- No response / connection error → network/environment issue in whatever context you're running from (relevant if this is happening only in the cloud routine, not locally — see step 4).

If you don't have `SLACK_WEBHOOK_URL` in hand, ask the user for it rather than guessing or reading it out of `.env`/git history into logs.

### 2. Did delivery-agent actually run, and was Slack even attempted?

Read `config/delivery.md` — confirm `Slack: yes` and that the env var name matches what delivery-agent checks (`SLACK_WEBHOOK_URL`). If Slack is disabled here, that alone explains a missing message and there's nothing to debug in code.

### 3. Local runs: the env-var-doesn't-persist trap

Each Bash tool call is a fresh shell (profile-initialized, but no memory of previous calls' `export`s). A one-off `export SLACK_WEBHOOK_URL=...` in one Bash call — yours or the user's — is invisible to the next call, including the one that actually runs `curl`. This has caused false "it's set" / silent-skip confusion before in this project. Check for this specifically:

- If testing manually, export and send in the **same** Bash invocation (`export ... && curl ...`), or
- Tell the user to add the export to their shell profile (`~/.zshrc`) permanently if they want ordinary local `/newsletter` runs to pick it up.

### 4. Cloud routine (scheduled daily run): check the routine itself before suspecting the code

Load `RemoteTrigger` (`ToolSearch select:RemoteTrigger`) and check the routine:

```
RemoteTrigger action:get trigger_id:<the routine's trig_... id>
```

Look at:
- `enabled` — if `false`, it's not running at all, period.
- `ended_reason` — `auto_disabled_init_failed` means the cloud **session itself never started** (e.g. couldn't clone the repo, environment problem). This has nothing to do with delivery-agent's logic — don't go debug the Slack code for this. Likely causes: the GitHub repo went private/permissions changed, or the configured `environment_id` lost repo access. Confirm the repo is reachable (`gh repo view <org>/<repo>`) and that the routine's `session_context.sources[0].git_repository.url` still matches. If it looks fine, re-enable (`action:update`, `{"enabled": true}`) and manually fire it (`action:run`) to get a fresh attempt, then re-check `get` after a few minutes (a full 6-agent pipeline run took ~16 minutes in this project's own test — don't check too early and conclude failure prematurely).
- If `enabled: true` and no `ended_reason`, but no new `history/<date>.md` shows up in the repo after enough time — the session started but the pipeline itself errored or stalled partway. That's when it becomes a code question (steps 5-6).

### 5. Did the pipeline even reach delivery-agent?

Per `.claude/commands/newsletter.md`, any stage returning empty/broken output stops the pipeline before delivery. Check:
- `git log --oneline -- history/` and `git pull && ls history/` — is there a `history/<today>.md`? Its absence after a completed cloud run means either delivery never ran, or it ran but the "commit and push the archive" step (in `newsletter.md`) failed separately from the Slack send itself — these are two different failure points, don't conflate them.
- If you have visibility into the run (local run in this same conversation, or the user can share cloud session output), check whether collector-agent found zero candidates, or dedup-agent filtered everything as already-delivered — either would legitimately produce no digest and no Slack message, with no bug involved.

### 6. Read the actual delivery code for a real bug

If steps 1-5 don't explain it, read `.claude/agents/delivery-agent.md` and check for logic issues: malformed JSON payload construction, wrong `curl` flags/headers, Slack Block Kit payload exceeding limits (50 blocks / 3000 chars per text field), or the temp-file-then-curl pattern not being followed (inline JSON in a Bash string is prone to quote-escaping bugs with special characters in article titles — this is exactly why the instructions specify writing to a temp file first). Validate any payload with `python3 -m json.tool <file>` before sending it.

## After fixing

- State plainly which layer failed and why (e.g. "the cloud routine's session never initialized — repo access issue, not a Slack bug" vs. "delivery-agent's payload had unescaped quotes from an article title").
- Re-verify with an actual end-to-end send, not just a code review — either trigger `/newsletter` locally with the webhook properly exported, or `RemoteTrigger action:run` the cloud routine and confirm a new `history/<date>.md` lands and the user visually confirms the Slack message.
- If the fix was to a `.claude/agents/*.md` or `.claude/commands/*.md` file, commit and push it — the cloud routine clones fresh each run and won't see local-only changes.
