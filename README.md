# Prompt Gym — Ralph's Daily Prompt Coach

> ### One-line setup (for OpenClaw agents)
> Say to your agent: **"Set up Prompt Gym from https://github.com/Freakingnolife/prompt-gym"**
> Your agent reads `START_HERE.md` and walks you through everything — one question at a time.

> **"You can't improve what you can't see."**
> Ralph reviews all your agent prompts every 24 hours, scores them against the OpenAI prompt guidance framework, and sends you a daily digest — automatically.

---

## What it does

Every day at a scheduled time, Ralph (your QA agent) wakes up and:

1. **Scans** all your agent sessions from the last 24 hours
2. **Filters** to only your direct prompts
3. **Scores** each one against the 9-pillar audit framework
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

## The 9-Pillar Audit Framework

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

The interactive setup handles most of this. You'll need:

- OpenClaw running with at least one agent
- A second agent configured as your QA coach (the agent doing the reviewing)
- A Discord account (for receiving digests)
- About 5 minutes to answer questions

---

## Quick Start

### The easy way: let your agent do it

Say to your OpenClaw agent:

> **"Set up Prompt Gym from https://github.com/Freakingnolife/prompt-gym"**

Your agent reads `SETUP.md` and walks you through everything — one question at a time. You just answer. It handles the cron jobs, skill install, and Discord channel creation.

### The manual way

1. Read `START_HERE.md` — understand what it does in 30 seconds
2. Read `INSTALL.md` — follow the step-by-step guide
3. Read `CONFIG.md` — customise the configuration
4. Read `SETUP.md` — if your agent is running the interactive setup
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
├── START_HERE.md          ← Entry point: 30-sec overview + how to begin
├── SETUP.md               ← Interactive setup (for your AI agent to run)
├── README.md              ← You are here
├── SKILL.md               ← The 9-pillar audit framework (for OpenClaw)
├── CRON_SETUP.md          ← Step-by-step cron job setup
├── CONFIG.md              ← Configuration reference
├── TEMPLATES.md           ← Digest output formats + prompt templates
├── INSTALL.md             ← Full manual installation guide
├── CLAWHUB.md             ← How to publish on ClawhHub
├── LICENSE                ← MIT
└── .github/
    └── workflows/
        └── prompt-gym-test.yml ← CI: lints skill on every push
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
