# Prompt Gym — 30-Second Overview

> **What it does:** Every morning, an AI coach reviews all your prompts from yesterday and tells you what to improve.
>
> **What you get:** A daily digest in Discord, like this:
> ```
> ✅ You did X well (3 times this week)
> ⛔ Fix this: add "Done when X" to your delegation prompts
> 🔴 Red flag: this prompt caused the agent to go off-track
> 📝 This week: practice one thing
> ```

---

## What You Need

| What | Details |
|------|---------|
| **OpenClaw** | Running with at least one agent |
| **A second agent** | Called "Ralph" — your QA coach (we'll set this up) |
| **Discord** | For receiving your daily digest |
| **5 minutes** | To get it running |

---

## How It Works

```
You send prompts → agents respond → Ralph watches everything
                                         ↓
                              Every day at 3am (your time)
                                         ↓
                              Ralph reviews yesterday's prompts
                                         ↓
                              Digest lands in your Discord DMs
```

---

## Ready to Set Up?

Say to your OpenClaw agent:

> "Set up Prompt Gym from https://github.com/Freakingnolife/prompt-gym"

Your agent will read the setup guide and walk you through it — one question at a time. You just answer.

---

## Or Follow the Steps Manually

1. Read `INSTALL.md` — full 15-minute setup
2. Read `SKILL.md` — what the coach actually does
3. Read `CONFIG.md` — all the things you can customise

---

## The 9 Pillars (What Ralph Scores)

| Pillar | What it means |
|--------|--------------|
| **Output Contract** | Did you say what format you wanted back? |
| **Scope & Boundaries** | Did you say when to stop? |
| **Tool & Dependencies** | Did you say what to check first? |
| **Task Classification** | Did you pick the right reasoning mode? |
| **Context Hygiene** | Did you give the right amount of context? |

---

## Want to Know More?

- `README.md` — full story and architecture
- `TEMPLATES.md` — the actual prompt templates Ralph uses
- `CLAWHUB.md` — how to publish on ClawhHub
