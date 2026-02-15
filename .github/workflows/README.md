# OpenClaw Management Workflows

GitHub Actions workflows for managing multi-user OpenClaw (moltworker) instances.

## Workflows

| Workflow | Trigger | Description |
|----------|---------|-------------|
| **Create New OpenClaw** | `workflow_dispatch` | Provisions a new `openclaw-<username>` worker with dedicated R2 bucket |
| **Update OpenClaw** | `workflow_dispatch` | Re-deploys code and updates secrets for an existing instance |
| **Delete OpenClaw** | `workflow_dispatch` | Deletes a worker and optionally its R2 bucket |

## Required GitHub Repository Secrets (8)

Configure at: **Settings > Secrets and variables > Actions > New repository secret**

| Secret | Description |
|--------|-------------|
| `CLOUDFLARE_API_TOKEN` | Wrangler CLI auth token. Permissions: Workers Scripts Edit, Workers R2 Storage Edit, Account Settings Read. [Create token](https://dash.cloudflare.com/profile/api-tokens) |
| `CF_ACCOUNT_ID` | Cloudflare Account ID (shared for wrangler CLI, AI Gateway, R2). [Find it](https://dash.cloudflare.com) > any domain > Overview (right sidebar) |
| `CLOUDFLARE_AI_GATEWAY_API_KEY` | Provider API key routed through AI Gateway |
| `CF_AI_GATEWAY_GATEWAY_ID` | AI Gateway ID. [Find it](https://dash.cloudflare.com) > AI > AI Gateway |
| `R2_ACCESS_KEY_ID` | R2 API token access key. [Manage tokens](https://dash.cloudflare.com) > R2 > Manage R2 API Tokens |
| `R2_SECRET_ACCESS_KEY` | R2 API token secret key |
| `CF_ACCESS_TEAM_DOMAIN` | Cloudflare Zero Trust team domain (e.g. `greybox.cloudflareaccess.com`). [Find it](https://one.dash.cloudflare.com/) > Settings > Custom Pages |
| `CF_ACCESS_AUD` | Access application audience tag from the Workers Access app |

## Auto-managed Secrets (no manual setup needed)

These are set automatically per worker by the workflows:

| Secret | Source |
|--------|--------|
| `MOLTBOT_GATEWAY_TOKEN` | Auto-generated (32-byte hex) by create workflow |
| `TELEGRAM_BOT_TOKEN` | Provided as workflow input at dispatch time |
| `TELEGRAM_DM_ALLOW_FROM` | Telegram user ID(s) for DM allowlist |
| `CF_AI_GATEWAY_MODEL` | Derived from `model` workflow input |
| `CF_AI_GATEWAY_ACCOUNT_ID` | Reuses `CF_ACCOUNT_ID` |
| `R2_BUCKET_NAME` | Auto-set to `openclaw-<username>-data` |
| `SANDBOX_SLEEP_AFTER` | Workflow input (default: `30m`) |
| `CDP_SECRET` | Auto-generated (32-byte hex) by create workflow |
| `WORKER_URL` | Auto-set to `https://openclaw-<username>.notifly.workers.dev` |

## Usage

### Create a new instance

1. Go to **Actions** > **Create New OpenClaw** > **Run workflow**
2. Enter:
   - `username`: lowercase alphanumeric, hyphens allowed (e.g. `alice`, `team-dev`)
   - `model`: AI Gateway model (dropdown). Options:
     - `openai/gpt-5.2` — OpenAI GPT-5.2
   - `instance_type`: container instance type (dropdown, default: `standard-1`). Options: `standard-1`, `standard-2`, `standard-3`, `standard-4`. See [Container limits](https://developers.cloudflare.com/containers/platform-details/limits/)
   - `telegram_bot_token`: Telegram bot token ([how to create](#creating-a-telegram-bot-token))
   - `telegram_user_id`: your Telegram user ID for DM allowlist ([how to find](#finding-your-telegram-user-id))
   - `sandbox_sleep_after`: container sleep timeout in minutes, or `never` (default: `30`)

This creates worker `openclaw-<username>` and R2 bucket `openclaw-<username>-data`.

### Update an existing instance

1. Go to **Actions** > **Update OpenClaw** > **Run workflow**
2. Enter:
   - `username`: existing instance username
   - `model`: new AI model (dropdown, default: `keep-existing`)
   - `instance_type`: container instance type (dropdown, default: `keep-existing`)
   - `telegram_bot_token`: (optional) leave empty to keep existing token
   - `telegram_user_id`: (optional) Telegram user ID for DM allowlist (leave empty to keep existing)
   - `sandbox_sleep_after`: (optional) container sleep timeout in minutes or `never` (leave empty to keep existing)

### Delete an instance

1. Go to **Actions** > **Delete OpenClaw** > **Run workflow**
2. Enter:
   - `username`: instance to delete

This deletes the worker, container, and R2 bucket (including all stored data).

## Finding Your Telegram User ID

1. Open Telegram and search for [@userinfobot](https://t.me/userinfobot)
2. Send any message (e.g. `/start`)
3. The bot replies with your numeric user ID (e.g. `123456789`)

Alternatively, use [@RawDataBot](https://t.me/RawDataBot) — send `/start` and look for the `id` field under `from`.

Multiple user IDs can be provided as a comma-separated list (e.g. `123456789,987654321`).

## Creating a Telegram Bot Token

1. Open Telegram and search for [@BotFather](https://t.me/BotFather)
2. Send the `/newbot` command
3. Enter a display name for the bot (e.g. `My OpenClaw`)
4. Enter a username — must end with `Bot` (e.g. `my_openclaw_bot`)
5. Copy the token issued by BotFather (format: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)

Treat the token like a password and never expose it publicly. If compromised, use `/revoke` in BotFather to regenerate.
