# Templates — Digest Formats and Prompt Templates

## Ralph's Digest Templates

### Full Digest (Standard — use this)

```
📊 PROMPT DIGEST — [date]

Volume: N interactions reviewed | last 24h
Agents: [list]

✅ WHAT YOU DID WELL
1. [Pattern name] (N instances)
   Example: "[truncated prompt]"
   → Outcome: [what happened because it worked]

[Max 3 good patterns]

---

🎯 TOP 3 PATTERNS TO IMPROVE
1. [Pattern name] (N of N prompts)
   What happened: [specific failure mode]
   Fix: [one-line fix]
   Example: "[truncated bad prompt]"

[Max 3 bad patterns]

---

📝 THIS WEEK: ONE THING TO PRACTICE

Before sending any multi-step delegation prompt, add:
"Done when: [criteria]. Stop when done."

Target: get this into 80% of your delegation prompts this week.

---

[🔴 RED FLAG — only if ≥2 instances]
[Specific prompt that caused a real problem]
What happened: [consequence]
Fix: [what to add next time]
```

---

### Compact Digest (Low-volume days — <5 prompts)

```
📊 PROMPT DIGEST — [date]

Volume: N prompts | last 24h

Quick take:
✅ [Best pattern this week]
⛔ [Biggest gap]
📝 Practice: [one thing]

Full digest when volume is higher.
```

---

### Empty / No Prompts Digest

```
📊 PROMPT DIGEST — [date]

No direct messages from you in the last 24h.
Nothing to review.

Start tomorrow morning. 🏁
```

---

## Prompt Templates — Use These Going Forward

### Simple Task
```
[TASK]
Done when: [completion criteria]
Stop when done — don't ask unless blocked.
```

### Multi-Step Task
```
[TASK DESCRIPTION]

Steps:
1. [step]
2. [step]
3. [step]

Done when: all steps complete
Output: [format spec — e.g. "table: task | owner | deadline"]
If blocked at any step: tell me exactly what's missing, then stop.
```

### Review/Audit Task
```
Review [target] for [criteria].
Return: one line per item. ✅ / ❌ / ⚠️
Do NOT make any changes.
End with: "X of Y issues [status]."
```

### Sub-agent handoff
```
Task: [description]
Workdir: [path]
Output: [format]
Constraints: [what NOT to do]
Agents to use: [which agents]
If blocked: [what to do]
```

### Plan-Only Request
```
Plan only — don't execute anything yet.
List 5 steps to accomplish: [task]
For each step: [what to do], [tool needed], [risk if wrong]
```

### Review + Improve Prompt (Meta)
```
Review this prompt: [paste prompt]
Against: OpenAI prompt guidance 9 pillars
Return: [format spec]
Fix: [what to add/remove/change]
```

---

## Quick-Reference Cheat Sheet

### Before Sending Any Prompt — Check These 5

| # | Check | If Missing |
|---|-------|------------|
| 1 | Output format specified? | Add: "Return: (1)... Max N bullets." |
| 2 | Done criteria defined? | Add: "Done when: [criteria]." |
| 3 | Stop rule clear? | Add: "Stop when done — don't ask unless blocked." |
| 4 | Prerequisites stated? | Add: "Check [memory/workspace/file] first." |
| 5 | Scope right-sized? | Cut excess context / add missing context |

### Most Common Fixes (One-Liners)

| Problem | Fix |
|---------|-----|
| Agent asks too many questions | Add: "Stop when done." |
| Output is too verbose | Add: "Max 3 bullets." |
| Agent assumes things | Add: "Only use [specific source]." |
| Agent changes things when reviewing | Add: "Review only — do not make changes." |
| Multi-step gets partial results | Add: "Complete all steps. Save to [file] if blocked." |
| Agent infers facts | Add: "Mark any inference as [inference] — only assert from [source]." |
| Wrong agent gets the task | Add: "Delegate to [agent name]." |

---

## Ralph's Scoring Sheet

Ralph scores each prompt 0-10:

| Pillar | 0 (no contract) | 1 (partial) | 2 (full) |
|--------|-----------------|-------------|-----------|
| 1. Output | No format | Format or length only | Format + length |
| 2. Scope | No done/stop | Done OR stop only | Done + stop |
| 3. Tools | No rules | 1 rule | 2+ rules |
| 4. Classification | Wrong mode | Implicit | Explicit |
| 5. Context | Over/under | Minor issue | Tight |

**Score → Signal:**
- 8-10: Excellent prompt
- 5-7: Good with gaps
- 3-4: Needs significant work
- 0-2: Likely to cause problems

Ralph flags patterns at 2+ instances. Scores alone don't appear in the digest — only actionable patterns.
