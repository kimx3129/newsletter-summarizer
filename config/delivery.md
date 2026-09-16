# Delivery Channels

delivery-agent attempts only the channels marked "yes" below, AND only if that channel's required environment variable(s) are actually set. A channel that's enabled here but missing its credentials is skipped with a note — not a pipeline failure. Edit this file directly to turn channels on/off; no code changes needed.

## Enabled today
- Slack: yes
- Discord: no
- Gmail: no
- Notion: no

## Slack
- **Primary: MCP connector.** Channel: `#daily-it-newsletter` (channel_id: `C0BG1V88AHL`, workspace: sungmin-workspace). Requires the Slack connector to be connected at https://claude.ai/customize/connectors and attached to whichever routine/session is running this pipeline. Cloud routine sandboxes block direct internet access to `hooks.slack.com`, so this is the only path that reliably works for scheduled runs — see delivery-agent.md for the exact tool call.
- **Fallback: Incoming Webhook.** Env var: `SLACK_WEBHOOK_URL`. POST a JSON payload (Block Kit or plain `text` with `mrkdwn`) to the webhook URL. Works for local/manual runs; expect it to fail in the cloud sandbox due to the egress block above.

## Discord (Incoming Webhook)
- Env var: `DISCORD_WEBHOOK_URL`
- POST JSON `{"content": "..."}` to the webhook URL. Discord's `content` field caps at ~2000 characters — if the digest is longer, split into multiple sequential POSTs (one per chunk, or one per story) rather than truncating content.

## Gmail (SMTP, via App Password)
- Env vars: `GMAIL_ADDRESS`, `GMAIL_APP_PASSWORD` (a Gmail **App Password**, not the real account password — generate one at https://myaccount.google.com/apppasswords, requires 2FA enabled on the account), `GMAIL_TO` (recipient; defaults to `GMAIL_ADDRESS` if unset)
- Send via `curl --url 'smtps://smtp.gmail.com:465' --ssl-reqd --mail-from "$GMAIL_ADDRESS" --mail-rcpt "$GMAIL_TO" --upload-file <raw-email-file> --user "$GMAIL_ADDRESS:$GMAIL_APP_PASSWORD"`, where `<raw-email-file>` is a plain file with `From:`/`To:`/`Subject:`/`Content-Type:` headers, a blank line, then the body.

## Notion (API)
- Env vars: `NOTION_API_KEY` (internal integration token), `NOTION_DATABASE_ID` (target database — must already be shared with the integration in Notion's UI, delivery-agent cannot do this part)
- Create one new page per run via `POST https://api.notion.com/v1/pages` with header `Notion-Version: 2022-06-28` and `Authorization: Bearer $NOTION_API_KEY`, `parent.database_id = NOTION_DATABASE_ID`, page title = `Tech/AI Digest — <date>`, body = digest content as block children (bulleted list per story).
