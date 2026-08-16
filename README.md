# Telegram JD to Notion

A small Cloudflare Worker that saves job descriptions from Telegram into a Notion database.

## How it works

1. Send a job description to your Telegram bot.
2. Gemini reads the text and extracts:
   - the company name
   - the job title
3. The Worker creates a new page in Notion.
4. The bot sends a success or error message back to Telegram.

The `Due date` field is set to the current date in Vietnam (`Asia/Ho_Chi_Minh`).

## Notion database

Create these properties with these exact names:

| Property | Recommended type | Value |
| --- | --- | --- |
| `Company` | Title or Text | Company name |
| `Status` | Status, Select, or Multi-select | `Not started` |
| `Due date` | Date | Date when the JD was sent |
| `Tags` | Text, Select, or Multi-select | Job title, such as `Software Engineer` |
| `Description` | Text | Full job description |

Share the database with your Notion integration. If `Tags` is a Select or Multi-select property, the integration also needs permission to add new options.

## Requirements

- Node.js
- A Cloudflare account
- A Telegram bot token
- A Gemini API key
- A Notion integration and database

## Create a Telegram bot with BotFather

1. Open Telegram and search for **@BotFather**.
2. Start the chat and send:

   ```text
   /newbot
   ```

3. Enter a display name for your bot, for example `Job Inbox`.
4. Enter a unique username ending in `bot`, for example `my_job_inbox_bot`.
5. BotFather will give you a token that looks like this:

   ```text
   123456789:AAExampleTokenHere
   ```

6. Keep this token private. You will use it as the `TELEGRAM_TOKEN` secret.

## Install

```bash
npm install
```

## Add secrets

Run these commands and enter the value when Wrangler asks:

```bash
npx wrangler secret put TELEGRAM_TOKEN
npx wrangler secret put NOTION_KEY
npx wrangler secret put NOTION_DB_ID
npx wrangler secret put GEMINI_KEY
```

`GEMINI_MODEL` is optional. The default model is `gemini-3.5-flash-lite`. The Worker automatically retries temporary Gemini errors and can fall back to `gemini-3.5-flash`.

Only set `GEMINI_MODEL` if you need a different model:

```bash
npx wrangler secret put GEMINI_MODEL
```

## Deploy

```bash
npm run deploy
```

After deployment, copy your Worker URL. It usually looks like:

```text
https://your-worker.your-subdomain.workers.dev
```

## Connect Telegram to the Worker

Set the Telegram webhook. Replace both placeholders:

```text
https://api.telegram.org/bot<TELEGRAM_TOKEN>/setWebhook?url=https://<WORKER_URL>
```

Open that URL in a browser or send it with `curl`:

```bash
curl "https://api.telegram.org/bot<TELEGRAM_TOKEN>/setWebhook?url=https://<WORKER_URL>"
```

The response should contain `"ok":true`.

## Use the bot

Send the complete JD as normal text. For example:

```text
Company: Example Company

Software Engineer Intern

Responsibilities:
- Build web applications
- Work with the engineering team

Requirements:
- JavaScript
- Basic English
```

The Worker will save:

- `Company`: `Example Company`
- `Tags`: `Software Engineer Intern`
- `Status`: `Not started`
- `Due date`: today in Vietnam
- `Description`: the complete message

## Important notes

- Send text or a Telegram caption. PDF, Word files, images, and voice messages are not processed yet.
- Send the JD in one message. Very long messages may be split by Telegram and create multiple Notion pages.
- Do not commit API keys or tokens. Use Wrangler secrets.
- A `GET` request to the Worker returns `Hello World!` as a simple health check.

## Development

```bash
npm run dev
```

Run type checking:

```bash
npx tsc --noEmit
```

Run tests:

```bash
npm test
```

## Troubleshooting

### Gemini returns HTTP 503

The model may be temporarily busy. The Worker retries automatically and tries the fallback model. Check the error message and try again later if both models are unavailable.

### Gemini returns HTTP 404

Check that `GEMINI_KEY` is a valid Gemini API key. If you set `GEMINI_MODEL`, use a model available to your account, or remove the override to use the default model.

### Tags is empty or fails

Check that the Notion property is named exactly `Tags`. For Select or Multi-select, share the database with the integration and give it permission to update the database schema.

### Notion returns an error

Check that:

1. The database is shared with the integration.
2. The property names match the table above.
3. `Due date` is a Date property.
4. `Description` is a Text property.
