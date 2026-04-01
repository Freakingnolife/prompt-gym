# Installation Guide — Full Setup in 15 Minutes

This guide walks you through setting up Ralph's Prompt Gym from scratch.

## Prerequisites

- OpenClaw installed and running
- At least one additional agent configured (Ralph — your QA agent)
- Discord bot account for Ralph
- Shared server between the bot and your Discord account
- `curl`, `python3`, `jq` available on your system

---

## Step 1: Configure Ralph's Discord Bot

### 1a: Get Ralph's Bot Token

```bash
grep OPENCLAW_DISCORD_RALPH_TOKEN ~/.openclaw/.env
```

If it doesn't exist, you'll need to create a Discord bot for Ralph:
1. Go to https://discord.com/developers/applications
2. Create application → "Ralph"
3. Bot → Reset Token → save it
4. Add to your OpenClaw config: `OPENCLAW_DISCORD_RALPH_TOKEN=<token>`

### 1b: Enable Necessary Intents

In Discord Developer Portal → Ralph's Bot → Bot settings:

- ✅ **PRESENCE INTENT** (recommended)
- ✅ **SERVER MEMBERS INTENT** (recommended)
- ✅ **MESSAGE CONTENT INTENT** (required)

### 1c: Add Ralph to Your Server

1. Discord Developer Portal → Ralph's Application → OAuth2 → URL Generator
2. Scopes: `bot`
3. Bot Permissions: `Send Messages`, `Create DMs`
4. Copy the generated URL → open in browser → select your server

---

## Step 2: Install the Skill

```bash
# Create the skill directory
mkdir -p ~/.openclaw/skills/prompt-review

# Copy the skill
cp prompt-gym/SKILL.md ~/.openclaw/skills/prompt-review/SKILL.md

# Verify
cat ~/.openclaw/skills/prompt-review/SKILL.md | head -20
```

---

## Step 3: Get Your Discord User ID

If you don't know your Discord user ID:

1. Discord settings → Advanced → Developer Mode: ON
2. Right-click your name → "Copy User ID"

Save it. You'll need it for the next steps.

---

## Step 4: Create Ralph's DM Channel

```bash
RALPH_TOKEN="your_ralph_bot_token_here"
YOUR_DISCORD_ID="your_discord_id_here"

curl -s -X POST \
  -H "Authorization: Bot $RALPH_TOKEN" \
  -H "Content-Type: application/json" \
  "https://discord.com/api/v10/users/$YOUR_DISCORD_ID/channels" \
  -d "{\"recipient_id\": \"$YOUR_DISCORD_ID\"}" | python3 -c "
import json,sys
d=json.load(sys.stdin)
print(f\"DM Channel ID: {d['id']}\")
print(f\"Save this: {d['id']}\")
"
```

Save the DM Channel ID — you'll need it in Step 6.

---

## Step 5: Calculate Your Next Run Time

```python
import datetime

# Example: 3am SGT tomorrow
sgt = datetime.datetime(2026, 4, 2, 3, 0, 0)
utc = sgt - datetime.timedelta(hours=8)
ms = int(utc.timestamp() * 1000)
print(f"nextRunAtMs: {ms}")
```

Adjust the `sgt` datetime to your actual first run time and timezone.

---

## Step 6: Add the Cron Job

```bash
# Backup existing jobs
cp ~/.openclaw/cron/jobs.json ~/.openclaw/cron/jobs.json.bak

# Add the job (replace values first!)
cat >> ~/.openclaw/cron/jobs.json.new << 'JOBEOF'
{
  "id": "prompt-gym-daily",
  "agentId": "ralph",
  "name": "Prompt Gym → Daily (3am SGT)",
  "enabled": true,
  "createdAtMs": 1775070000000,
  "updatedAtMs": 1775070000000,
  "schedule": {
    "kind": "cron",
    "expr": "0 3 * * *",
    "tz": "Asia/Singapore"
  },
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "model": "anthropic/claude-sonnet-4-6",
    "timeoutSeconds": 300,
    "message": "Read ~/.openclaw/skills/prompt-review/SKILL.md then run the prompt gym. Deliver to Discord DM channel YOUR_DM_CHANNEL_ID using curl with Ralph's token (OPENCLAW_DISCORD_RALPH_TOKEN env var)."
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "accountId": "ralph",
    "to": "YOUR_DM_CHANNEL_ID",
    "bestEffort": true
  },
  "state": {
    "nextRunAtMs": 1775070000000,
    "lastRunAtMs": null,
    "lastRunStatus": null,
    "lastStatus": null,
    "lastDurationMs": null,
    "lastDeliveryStatus": null,
    "consecutiveErrors": 0,
    "lastDelivered": null
  }
}
JOBEOF

# Replace placeholders
sed -i '' 's/YOUR_DM_CHANNEL_ID/1234567890/g' ~/.openclaw/cron/jobs.json
# (Replace 1234567890 with your actual DM channel ID from Step 4)
```

---

## Step 7: Test It

Send Ralph's DM to yourself manually to confirm the pipeline works:

```bash
curl -s -X POST \
  -H "Authorization: Bot $OPENCLAW_DISCORD_RALPH_TOKEN" \
  -H "Content-Type: application/json" \
  "https://discord.com/api/v10/channels/YOUR_DM_CHANNEL_ID/messages" \
  -d '{"content": "Ralph'\''s Prompt Gym is set up. First daily digest goes out tomorrow at 3am SGT. 🏁"}'
```

---

## Step 8: Trigger a Live Test

In your Ralph agent session (or via the main agent):

```
Run the prompt gym for today as a test. Be rigorous.
```

Ralph will scan the last 24h of sessions and send a digest to your Discord DMs.

---

## Uninstall

```bash
# Remove the cron job
# Edit ~/.openclaw/cron/jobs.json and remove the job with id "prompt-gym-daily"

# Remove the skill
rm -rf ~/.openclaw/skills/prompt-review

# Restart gateway
openclaw gateway restart
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| "Unknown Channel" when sending DM | DM channel doesn't exist — run Step 4 again |
| "Missing Access" when sending DM | the bot not in any shared server with the user |
| Cron never fires | Check `nextRunAtMs` is in the future |
| Digest empty | No the user prompts in last 24h — check sessions are being tracked |
| Ralph times out | Increase `timeoutSeconds` to 600; reduce agents in `AGENTS_TO_SCAN` |

---

## Getting Help

- OpenClaw docs: https://docs.openclaw.ai
- GitHub issues: https://github.com/Freakingnolife/prompt-gym/issues
- Discord community: https://discord.com/invite/clawd
