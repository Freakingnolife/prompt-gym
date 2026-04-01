# Cron Setup — How to Schedule Ralph's Daily Digest

## Overview

The prompt digest runs as an OpenClaw cron job. The job wakes Ralph up at the scheduled time, Ralph reads the SKILL.md, runs the audit, and delivers the digest to Discord.

## Step-by-Step Setup

### Step 1: Get Ralph's DM Channel ID

Ralph needs a Discord DM channel to send messages to. To create/find the channel ID:

```bash
# Using Ralph's bot token — creates DM channel with Marcus
curl -s -X POST \
  -H "Authorization: Bot $OPENCLAW_DISCORD_RALPH_TOKEN" \
  -H "Content-Type: application/json" \
  "https://discord.com/api/v10/users/MARCUS_DISCORD_ID/channels" \
  -d '{"recipient_id": "MARCUS_DISCORD_ID"}'

# Response: { "id": "CHANNEL_ID", "type": 1, ... }
# Use the "id" field as YOUR_DM_CHANNEL_ID
```

Save the channel ID — you'll need it for the cron job.

### Step 2: Add the Cron Job

Add to your `~/.openclaw/cron/jobs.json`:

```json
{
  "id": "prompt-digest-daily",
  "agentId": "ralph",
  "name": "Prompt Digest → Daily (3am SGT)",
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
    "message": "Read ~/.openclaw/skills/prompt-review/SKILL.md then run the prompt digest for Marcus. Deliver to Discord DM channel 1478943974910853293 using curl with Ralph's token (OPENCLAW_DISCORD_RALPH_TOKEN env var)."
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
```

### Step 3: Update the channel ID

Replace `"to": "YOUR_DM_CHANNEL_ID"` with the actual channel ID from Step 1.

### Step 4: Set the next run time

Calculate the correct `nextRunAtMs` for your schedule:

```python
import datetime

# Example: 3am SGT tomorrow
sgt = datetime.datetime(2026, 4, 2, 3, 0, 0)
utc = sgt - datetime.timedelta(hours=8)
ms = int(utc.timestamp() * 1000)
print(ms)  # 1775070000000
```

### Step 5: Verify

Check the job was added:

```bash
cat ~/.openclaw/cron/jobs.json | python3 -c "
import json,sys
d=json.load(sys.stdin)
job = [j for j in d['jobs'] if 'prompt-digest' in j.get('id','')]
print(json.dumps(job, indent=2))
"
```

---

## Schedule Options

| Schedule | Cron expression | Use case |
|----------|-----------------|----------|
| Daily 3am | `0 3 * * *` | Most users — review yesterday's work |
| Daily 8am | `0 8 * * *` | Morning before work |
| Weekly Sunday 9pm | `0 21 * * 0` | Bulk review, less frequent |
| Every weekday 8am | `0 8 * * 1-5` | Workdays only |

Change `"tz"` to your timezone (IANA timezone database names, e.g. `America/New_York`).

---

## Testing the Cron

To trigger a test run immediately:

1. OpenClaw cron jobs have a `wakeMode` — set to `"now"` temporarily
2. Or manually spawn Ralph with the digest task
3. Or use the OpenClaw CLI: `openclaw cron trigger <job-id>`

---

## Troubleshooting

### Ralph's DM doesn't arrive
1. Check Ralph's Discord bot has `CREATE_DM` access — test with:
   ```bash
   curl -s -X POST \
     -H "Authorization: Bot $OPENCLAW_DISCORD_RALPH_TOKEN" \
     -H "Content-Type: application/json" \
     "https://discord.com/api/v10/users/MARCUS_ID/channels" \
     -d '{"recipient_id": "MARCUS_ID"}'
   ```
   If this returns an error, fix Discord bot permissions first.

2. Check the `delivery.to` field matches the DM channel ID (not Marcus's user ID)

3. Check the cron job's `nextRunAtMs` — if it's in the past, update it

### Ralph times out
- Increase `timeoutSeconds` in the payload (try 600 instead of 300)
- Reduce `AGENTS_TO_SCAN` to fewer agents
- Reduce `LOOKBACK_HOURS` from 25 to 12

### Digest is empty
- Check Marcus actually sent direct messages in the last 24h
- Check the `sender_id` filter matches Marcus's actual Discord user ID
- Verify sessions are being tracked (check OpenClaw session history)

---

## Uninstalling

Remove the cron job:

```bash
# Edit jobs.json and remove the job with id "prompt-digest-daily"
# Then restart the gateway:
openclaw gateway restart
```
