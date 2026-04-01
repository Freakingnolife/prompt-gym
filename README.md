# Prompt Gym — Ralph's Daily Prompt Coach

> **"You can't improve what you can't see."**
> Ralph reviews all your agent prompts every 24 hours, scores them against the OpenAI prompt guidance framework, and sends you a daily digest — automatically.

---

## What it does

Every day at a scheduled time, Ralph (your QA agent) wakes up and:

1. **Scans** all your agent sessions from the last 24 hours
2. **Filters** to only your direct prompts
3. **Scores** each one against the 5-pillar audit framework
4. **Identifies** patterns — recurring mistakes and consistent wins
5. **Delivers** a structured digest to your Discord DMs

The digest looks like this:

```
📊 PROMPT DIGEST — Apr 1, 2026

✅ WHAT YOU DID WELL
1. Output contracts (3 instances) — agents structured output correctly
   Example: "...Format: one line per item. ✅/❌/⚠️"

🎯 TOP 3 PATTERNS TO IMPROVE
1. No done/stop rules (7 of 14 prompts)
   Fix: "Done when: [criteria]. Stop when done."

📝 THIS WEEK: ONE THING TO PRACTICE
"Done when: X. Stop when done." — target 80% of delegation prompts.

🔴 RED FLAG
[Specific prompt that caused a real problem]
```

---

## The 5-Pillar Audit Framework

Every prompt is scored on:

| Pillar | What it measures |
|--------|-----------------|
| **1. Output Contract** | Did you specify format, structure, length? |
| **2. Scope & Boundaries** | Did you define "done"? Stop rules? |
| **3. Tool & Dependencies** | Did you specify prerequisites? Block conditions? |
| **4. Task Classification** | Did you pick the right reasoning mode? |
| **5. Context Hygiene** | Was the prompt appropriately scoped? |

---

## Prerequisites

- OpenClaw running with multiple agents
- Ralph agent configured (your QA agent)
- Discord bot account for Ralph (with DM permissions)
- Discord server shared between Ralph's bot and you

---

## Quick Start

### Step 1: Install the skill

Copy `SKILL.md` to your OpenClaw skills directory:

```bash
cp SKILL.md ~/.openclaw/skills/prompt-review/SKILL.md
```

### Step 2: Configure

Edit the skill's CONFIG section:
- `RALPH_DISCORD_USER_ID` — your Discord user ID
- `RALPH_DM_CHANNEL_ID` — Ralph's DM channel ID
- `SCHEDULE` — cron expression (default: `0 3 * * *` = 3am daily)
- `AGENTS_TO_SCAN` — which agents to review (default: all)

### Step 3: Set up the cron job

Add to your `~/.openclaw/cron/jobs.json`:

```json
{
  "id": "prompt-gym-daily",
  "agentId": "ralph",
  "name": "Prompt Gym → Daily (3am)",
  "schedule": {
    "kind": "cron",
    "expr": "0 3 * * *",
    "tz": "Asia/Singapore"
  },
  "sessionTarget": "isolated",
  "wakeMode": "now",
  "payload": {
    "kind": "agentTurn",
    "timeoutSeconds": 300,
    "message": "Read SKILL.md then run the prompt gym for the user."
  },
  "delivery": {
    "mode": "announce",
    "channel": "discord",
    "accountId": "ralph",
    "to": "YOUR_DM_CHANNEL_ID"
  }
}
```

See `CRON_SETUP.md` for full step-by-step instructions.

### Step 4: Test it

```bash
# Run the digest manually right now
# In your Ralph agent session:
/run prompt-gym --test
```

---

## File Structure

```
prompt-gym/
├── README.md              ← You are here
├── SKILL.md               ← The 5-pillar audit framework (for OpenClaw)
├── CRON_SETUP.md          ← Step-by-step cron job setup
├── CONFIG.md              ← Configuration reference
├── TEMPLATES.md           ← Digest output formats + prompt templates
├── INSTALL.md             ← Full installation guide
├── .github/
│   └── workflows/
│       └── digest-test.yml ← Test the digest on your prompts (CI)
└── CLAWHUB.md             ← How to publish on ClawhHub
```

---

## Customising Ralph's Voice

Ralph is configured to be **rigorous and challenging** — not soft. Edit the skill's voice directives:

```markdown
Be rigorous. If a prompt is vague, say so. If a pattern is risky, 
flag it as a red flag. Do not soften the critique.
```

Change to suit your preference:
- **Encouraging:** "Good pattern. Keep doing this."
- **Minimal:** Just the numbers, no commentary.
- **Educational:** Include explanations of WHY each pattern matters.

---

## Credits

Built for the [OpenClaw](https://openclaw.ai) ecosystem.

The audit framework is adapted from [OpenAI's Prompt Guidance guide](https://developers.openai.com/api/docs/guides/prompt-guidance).
