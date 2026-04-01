# Configuration Reference

All configurable values for the Prompt Digest system.

## Cron Job Config (`~/.openclaw/cron/jobs.json`)

```json
{
  "id": "prompt-digest-daily",
  "agentId": "ralph",
  "name": "Prompt Digest → Daily (3am SGT)",
  "schedule": {
    "kind": "cron",
    "expr": "0 3 * * *",
    "tz": "Asia/Singapore"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `schedule.expr` | Yes | Cron expression. [Crontab guru](https://crontab.guru) for testing. |
| `schedule.tz` | Yes | IANA timezone name. Default: `Asia/Singapore` |
| `sessionTarget` | Yes | Use `"isolated"` for fresh session each run |
| `wakeMode` | Yes | Use `"now"` to fire immediately at scheduled time |
| `payload.timeoutSeconds` | Yes | Max runtime. 300s = 5 min. Increase if scanning many agents. |
| `payload.model` | No | Model for Ralph to use. Default: `anthropic/claude-sonnet-4-6` |
| `delivery.accountId` | Yes | Which Discord bot account sends the DM. Use `"ralph"`. |
| `delivery.to` | Yes | Discord DM channel ID, not user ID. |

---

## Environment Variables

Set in your OpenClaw environment or the skill context:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENCLAW_DISCORD_RALPH_TOKEN` | Yes | — | Ralph's Discord bot token. From `~/.openclaw/.env`. |
| `TARGET_USER_ID` | Yes | — | Your Discord user ID (the person receiving digests) |
| `DM_CHANNEL_ID` | Yes | — | Ralph's DM channel with you. Created via Discord API. |
| `LOOKBACK_HOURS` | No | `25` | Hours to scan back. Slightly >24 to avoid edge cases. |
| `MIN_PROMPTS_TO_FLAG` | No | `2` | Min instances before flagging a pattern |
| `MAX_GOOD_PATTERNS` | No | `3` | Max good patterns to show in digest |
| `MAX_BAD_PATTERNS` | No | `3` | Max bad patterns to show in digest |
| `DIGEST_TIMEZONE` | No | `Asia/Singapore` | Your timezone for display |

---

## Agents to Scan

Configure which agents Ralph reviews. Set in the cron payload's `message`:

```
Agents to check: main, dev, research, yf_erp, ayi, marketing, engineering, sales, ralph, formmate, chiefofstaff
```

| Agent | Purpose | Include? |
|-------|---------|----------|
| `main` | Primary agent, most prompts | ✅ Always |
| `dev` | Engineering / code | ✅ Recommended |
| `research` | Research tasks | ✅ Recommended |
| `yi_erp` | ERP / business data | ✅ If used |
| `ayi` | Home/life admin | ✅ If used |
| `marketing` | Marketing content | ✅ If used |
| `sales` | Sales outreach | ✅ If used |
| `ralph` | QA agent | ⚠️ Only if Ralph prompts others |
| `formmate` | Formlabs work | ✅ If relevant |
| `chiefofstaff` | Coordination | ✅ If used |

---

## Discord Setup

### Getting Ralph's DM Channel ID

```bash
# 1. Get Ralph's bot token from ~/.openclaw/.env
# OPENCLAW_DISCORD_RALPH_TOKEN=<your-bot-token>

# 2. Create/find DM channel with Marcus
curl -s -X POST \
  -H "Authorization: Bot $OPENCLAW_DISCORD_RALPH_TOKEN" \
  -H "Content-Type: application/json" \
  "https://discord.com/api/v10/users/MARCUS_DISCORD_ID/channels" \
  -d '{"recipient_id": "MARCUS_DISCORD_ID"}'

# Response: { "id": "1234567890", "type": 1, ... }
# Use the "id" value as DM_CHANNEL_ID
```

### Ralph's Discord Account Config

```
Account ID: ralph
Token env var: OPENCLAW_DISCORD_RALPH_TOKEN
Shared server: Your Discord server (wherever your bot and user share a guild)
Bot intent: CREATE_DM (enabled by default)
```

### Discord Rate Limits

- DM channels: 1 message per second per channel
- Ralph's digest is 2-4 messages → space them by 2s minimum
- Add `sleep 2` between curl calls if sending multiple messages

---

## Customisation Checklist

Before publishing or sharing:

- [ ] Replace `MARCUS_DISCORD_ID` with your actual Discord user ID
- [ ] Replace `DM_CHANNEL_ID` with your actual DM channel ID
- [ ] Update `schedule.tz` to your timezone
- [ ] Update `AGENTS_TO_SCAN` list to match your actual agents
- [ ] Update Ralph's voice directive if you want softer/harsher tone
- [ ] Set `DIGEST_TIMEZONE` to your timezone
- [ ] Update `README.md` credits (author name, GitHub handle)
