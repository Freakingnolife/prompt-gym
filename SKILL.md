---
name: prompt-review
description: Prompt coaching skill. Use when auditing prompts against the OpenAI prompt guidance framework. Reads all agent sessions, scores the user's prompts on the 5 pillars, produces a digest. Config: SCHEDULE, AGENTS_TO_SCAN, TARGET_USER_ID in the cron payload.
---

# Prompt Review Skill — Ralph's Daily Coaching Framework

> Ralph's 5-pillar audit framework. Scores every prompt against the OpenAI prompt guidance. Adapted for OpenClaw multi-agent setups.

## Core Loop

```
1. Find all sessions (all agents, last 24h)
2. Filter to the user's direct messages only (sender_id match)
3. Fetch full message history per session
4. Score each user's prompt against 5 pillars
5. Identify patterns (≥2 instances required)
6. Produce digest
7. Deliver to Discord DM
```

## The 5 Pillars

### Pillar 1 — Output Contract

**Did the prompt specify:**
- Format? (table, bullet list, one-liner, prose)
- Structure? (sections, numbered steps)
- Length? (max bullets, max words)

**Scoring:**
- ✅ Full contract (format + length) = 2pts
- ⚠️ Partial contract (format OR length) = 1pt
- ❌ No contract = 0pt

**Common failure:** "Tell me about X" → agent returns 10 paragraphs when you needed 3 bullets.

**Fix template:**
```
Return: (1) [what], (2) [what]. Max 3 bullets each.
```

---

### Pillar 2 — Scope & Boundaries

**Did the prompt define:**
- "Done" criteria? (what success looks like)
- Stop rules? (when to stop without asking)
- Explicit redirects? (when changing direction mid-conversation)

**Scoring:**
- ✅ Done criteria + stop rule = 2pts
- ⚠️ Done criteria OR stop rule = 1pt
- ❌ Neither = 0pt

**Common failure:** Agent asks for confirmation after completing simple tasks. Or keeps working on stale instructions after you've moved on.

**Fix template:**
```
Done when: [criteria].
Stop when done — don't ask unless blocked.
```

**Scoped redirect:**
```
Stop previous task. New task: [X].
Don't touch [Y].
```

---

### Pillar 3 — Tool & Dependency Rules

**Did the prompt specify:**
- Prerequisites? (check memory, check workspace first)
- Dependency order? (step 1 must complete before step 2)
- Block conditions? (if X, stop and tell me)
- Partial work rules? (if you can't finish, save state to [file])

**Scoring:**
- ✅ 2+ dependency rules = 2pts
- ⚠️ 1 rule = 1pt
- ❌ No rules = 0pt

**Common failure:** Agent skips memory retrieval, assumes facts, or stops after partial completion.

**Fix template:**
```
Before acting: check memory/workspace for prior context.
If blocked at any step: stop and tell me exactly what's missing.
Complete all steps or save partial work to [file].
```

---

### Pillar 4 — Task Classification

**Did the prompt pick the right reasoning mode?**

| Mode | When to use | What to add |
|------|-------------|-------------|
| Quick confirm | Lookup, simple yes/no | "Quick answer — no reasoning needed." |
| Standard | Normal delegation | Default — no spec needed |
| Research | Multi-source, open-ended | "Research mode: cover all angles." |
| Autonomy | Long implementation | "Persist through to completion unless paused." |
| Verification | High-stakes, irreversible | "Verification loop before finalizing." |

**Scoring:**
- ✅ Appropriate mode explicitly stated = 2pt
- ⚠️ Implicitly appropriate = 1pt
- ❌ Mismatched (complex task, no mode; or simple task over-specified) = 0pt

**Common failure:** Complex multi-step task given no mode guidance → agent uses quick-look mode and misses depth.

---

### Pillar 5 — Context Hygiene

**Was the prompt appropriately scoped?**
- Not too short (missing critical context)
- Not too long (confusing the agent with irrelevant info)
- Stale context purged? (old instructions that still apply but shouldn't)

**Scoring:**
- ✅ Tightly scoped to this task = 2pt
- ⚠️ Minor over/under-scope = 1pt
- ❌ Clearly too much or too little = 0pt

**Common failure:** Prompt includes old context that conflicts with new instructions.

**Fix template:**
```
Only consider: [current task].
Ignore: [anything that's no longer relevant].
```

---

## Pattern Rules

1. **Minimum 2 instances** to flag as a pattern — one-off observations aren't patterns
2. **Flag the actual text** — quote the prompt (truncated at 120 chars), don't summarise
3. **Be specific about impact** — what happened because of the gap, not just that a gap exists
4. **One recommended fix per pattern** — specific, actionable, from the fix templates above
5. **One "thing to practice" per week** — not a list of 10, pick the one highest-leverage change

---

## Digest Output Format

```
📊 PROMPT DIGEST — [date]

Volume: N interactions reviewed | last 24h
Agents: [list of agents user used]

✅ WHAT YOU DID WELL
1. [Pattern name] (N instances)
   Example: "[truncated prompt]"
   → Outcome: [what happened because it worked]

[Repeat for each good pattern — max 3]

---

🎯 TOP 3 PATTERNS TO IMPROVE

1. [Pattern name] (N of N prompts)
   What happened: [specific failure mode]
   Fix: [one-line fix]
   Example: "[truncated bad prompt]"

[Repeat for each pattern — max 3]

---

📝 THIS WEEK: ONE THING TO PRACTICE

Before sending any multi-step delegation prompt, add:
"Done when: [criteria]. Stop when done."

Target: get this into 80% of your delegation prompts this week.

---

🔴 RED FLAG (only if ≥2 instances across prompts — otherwise omit this section)

[Specific prompt that caused a real problem]
What happened: [consequence]
Fix: [what to add next time]
```

---

## Ralph's Voice Directive

Ralph is QA agent who challenges things. Edit as preferred:

> Be rigorous. If a prompt is vague, say so. If a pattern is risky, flag it as a red flag. Do not soften the critique.

Alternative voices:
- **Encouraging:** "Good instinct here — agents responded well. Keep doing this."
- **Educational:** "The gap here caused X because Y. Here's why that matters."
- **Minimal:** Just the pattern, count, and fix. No commentary.

---

## Configuration

Set in the cron job payload's environment or as skill variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `SCHEDULE` | `0 3 * * *` | Cron expression (3am daily) |
| `TZ` | `Asia/Singapore` | Your timezone |
| `AGENTS_TO_SCAN` | `main,dev,research,yf_erp,ayi,marketing` | Comma-separated agent names |
| `TARGET_USER_ID` | the user's Discord ID | Who to deliver the digest to |
| `LOOKBACK_HOURS` | `25` | How far back to scan (slightly >24 to avoid edge cases) |

---

## Anti-Patterns to Catch

These are the most common failure patterns Ralph flags:

| Pattern | Detection | Fix |
|---------|-----------|-----|
| No output format | Agent returns prose when you needed bullets | Add: "Return: (1)... Max N bullets." |
| No stop rule | Agent asks for confirmation after simple tasks | Add: "Stop when done — don't ask unless blocked." |
| Partial completion | Agent stops after step 1 of 2 | Add: "Complete ALL steps. Save state to [file] if blocked." |
| Wrong agent for task | Task given to wrong agent (e.g., Dev asked to research) | Add: "Delegate to [agent]." |
| No scope on skill-builds | Agent builds something without knowing the deliverable format | Add: "Build a [type]. Output goes to [where]. Reviewed [frequency]." |
| Executing when asked to review | Agent changes code when asked to audit | Add: "Review only — report findings. Do not make changes." |

---

## When NOT to Flag

- Quick confirmations ("yes", "ok", "got it") — these are fine as-is
- One-off gaps in otherwise good prompts — not a pattern without ≥2 instances
- Agent failures caused by tool errors (not prompt structure)
- Tasks where the agent was clearly blocked by external factors (network, API, etc.)

---

## Skill Usage

### As a live review (on-demand)
```
Run the prompt gym for today.
```

### As a scheduled coach (daily cron)
Scheduled via OpenClaw cron — see `CRON_SETUP.md`.

### As a template for building prompt skills
Use the 5-pillar framework to audit any prompt system. The framework is:
1. Actionable — each pillar has a specific fix template
2. Measurable — scores 0-2 per pillar, 10 points total
3. Pattern-based — minimum instances required to flag
4. Output-structured — digest format is consistent and scannable
