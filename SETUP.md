# Prompt Gym — Interactive Setup Guide

> For the agent: read this file when the user asks to set up Prompt Gym. Walk through each step one at a time. Ask the user one question, confirm their answer, then move to the next. Do not rush ahead.

---

## What You Need From the User (Collect These First)

Before generating any config, collect:

1. **Timezone** — e.g. Asia/Singapore, America/New_York, Europe/London
2. **Preferred time** — default is 3am (quiet time, before the day starts)
3. **Which agents to scan** — list the agent names they use
4. **Whether they have a Discord bot for their QA agent** — yes/no
5. **The QA agent's Discord bot token** — if they have one

---

## Step-by-Step Flow

### STEP 1 — Introduce Prompt Gym

Say to the user:

> "Prompt Gym is a daily AI prompt coach. Every morning it reviews all the prompts you sent to your agents yesterday, scores them, and sends you a digest of what to improve. Want me to set it up for you?"

**If no →** Stop. No setup needed.

**If yes →** Continue to Step 2.

---

### STEP 2 — Check Prerequisites

Ask:

> "Do you have a second OpenClaw agent set up as your QA coach? (This is separate from your main agent — it's the one that reviews your work.)"

**If yes →** Ask: "What's the agent's name?" → record it as `QA_AGENT_NAME`

**If no →** Say: "You'll need to create a second agent first. In OpenClaw, go to your agents config and add a new agent with a QA personality. Once you've done that, come back and say 'Prompt Gym is ready.'"

---

### STEP 3 — Timezone and Schedule

Ask:

> "What timezone are you in? And what time do you want the daily digest to arrive? (Default is 3am in your timezone — a good time to review before the day starts.)"

Record:
- `TIMEZONE` = e.g. Asia/Singapore
- `HOUR` = e.g. 3
- `CRON_EXPR` = `0 {HOUR} * * *`

---

### STEP 4 — Which Agents to Scan

Ask:

> "Which OpenClaw agents do you use most? (e.g. main, dev, research, marketing) — I'll scan prompts from these agents. You can list as many as you like, separated by commas."

Record: `AGENTS_TO_SCAN` = comma-separated list

---

### STEP 5 — Discord Setup

Ask:

> "Do you have a Discord bot account for your QA agent? (This is needed so the digest can be sent to your Discord DMs.)"

**If yes →** Ask: "Do you have the bot's token? (It looks like `MTQ3ODk0...` — you'll find it in Discord Developer Portal → Your App → Bot → Token.)"

**If no →** Say: "No problem. I'll give you instructions to create one — it takes about 2 minutes."

---

### STEP 6 — Create the Discord DM Channel

Once they have a bot token:

> "I need to create a Discord DM channel so your QA agent can send you messages. I'll do this automatically using your bot token."

Run this curl command (replace `BOT_TOKEN` and `YOUR_DISCORD_ID`):

```bash
curl -s -X POST \
  -H "Authorization: Bot BOT_TOKEN" \
  -H "Content-Type: application/json" \
  "https://discord.com/api/v10/users/YOUR_DISCORD_ID/channels" \
  -d '{"recipient_id": "YOUR_DISCORD_ID"}'
```

Ask the user for their Discord user ID if they don't know it. Tell them:
> "In Discord: Settings → Advanced → Developer Mode → ON. Then right-click your name and 'Copy User ID.'"

Record the `DM_CHANNEL_ID` from the curl response.

---

### STEP 7 — Build the Cron Job Config

Once you have all the values, generate this JSON (fill in the blanks):

```json
{
  "id": "prompt-gym-daily",
  "agentId": "{QA_AGENT_NAME}",
  "name": "Prompt Gym → Daily ({HOUR}:00 {TIMEZONE})",
  "enabled": true,
  "schedule": {
    "kind": "cron",
    "expr": "{CRON_EXPR}",
    "tz": "{TIMEZONE}"
  },
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 300,
    "message": "Read ~/.openclaw/skills/prompt-gym/SKILL.md then run the Prompt Gym daily digest. Deliver the digest to Discord DM channel {DM_CHANNEL_ID} using curl with the QA agent's Discord token (OPENCLAW_DISCORD_RALPH_TOKEN env var)."
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "accountId": "{QA_AGENT_NAME}",
    "to": "{DM_CHANNEL_ID}",
    "bestEffort": true
  }
}
```

Show this to the user and say:

> "Here's the configuration. Shall I add this to your cron jobs? (Say yes and I'll write it to ~/.openclaw/cron/jobs.json right now.)"

---

### STEP 8 — Install the Skill

Say:

> "I need to save the Prompt Gym skill to your OpenClaw skills folder. Give me a moment."

Run:
```bash
mkdir -p ~/.openclaw/skills/prompt-gym
# Then copy the SKILL.md content to ~/.openclaw/skills/prompt-gym/SKILL.md
```

Confirm: "Skill installed ✓"

---

### STEP 9 — Test It

Say:

> "Done! Your Prompt Gym digest will run every day at {HOUR}:00 {TIMEZONE}. Want me to trigger a test run now so you can see what it looks like?"

**If yes →** Run the digest manually now and send the result to the user's Discord DMs.

**If no →** Say: "No problem. The first digest will arrive tomorrow at {HOUR}:00 {TIMEZONE}."

---

### STEP 10 — Confirm Everything

Say:

> "**Prompt Gym is live.** Here's what you need to know:
>
> 📅 **First digest:** tomorrow at {HOUR}:00 {TIMEZONE}
> 📍 **Where:** Discord DMs from your QA agent
> 🔍 **What it reviews:** all your agent prompts from the last 24h
> ⏰ **Schedule:** every day at {HOUR}:00 {TIMEZONE}
>
> To change anything later:
> - Time: update the cron expression in ~/.openclaw/cron/jobs.json
> - Agents to scan: edit the skill message in the cron job
> - Voice/style: edit ~/.openclaw/skills/prompt-gym/SKILL.md
>
> That's it. You're done."

---

## Error Handling

| Problem | What to say |
|---------|-------------|
| User doesn't know their Discord user ID | Guide them to: Discord Settings → Advanced → Developer Mode → ON → right-click name → Copy User ID |
| User doesn't have a Discord bot | Give them the 2-minute Discord Developer Portal guide from `CONFIG.md` |
| Bot can't create DM | "Your bot needs to be in at least one server with you. Add it to any server you share." |
| Cron job fails | "Check ~/.openclaw/cron/jobs.json — the nextRunAtMs timestamp might be in the past. I can fix that." |
| User confused at any step | Break down the answer further. No jargon. |

---

## What NOT to Do

- ❌ Don't skip ahead — one question at a time
- ❌ Don't use jargon without explaining it
- ❌ Don't generate the cron JSON until you have ALL required values
- ❌ Don't touch the user's files without confirming first
- ❌ Don't assume they know what a cron expression is — it's normal to not know
