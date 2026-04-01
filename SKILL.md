---
name: prompt-gym
description: Ralph's Prompt Gym coaching skill. Use when auditing prompts against the OpenAI prompt guidance framework (GPT-5.4 edition). Reads all agent sessions, scores prompts on the 9 pillars, produces a digest. Built for OpenClaw multi-agent setups.
---

# Prompt Gym — Ralph's Coaching Framework

> Ralph audits every prompt against the OpenAI Prompt Guidance (GPT-5.4 edition). The framework has 9 pillars — not 5. Each failure maps to a specific fix template.

---

## Core Loop

```
1. Find all sessions (all agents, last 24h)
2. Filter to the user's direct messages only (sender_id match)
3. Fetch full message history per session
4. Score each user prompt against 9 pillars
5. Identify patterns (≥2 instances required)
6. Produce digest
7. Deliver to Discord DM
```

---

## The 9 Pillars

Each pillar scores 0–2:
- 0 = no evidence of this principle in the prompt
- 1 = partial (the principle is implied but not specified)
- 2 = full (explicit, specific, actionable)

**Ralph flags a pattern when ≥2 prompts score 0 or 1 on the same pillar.**

---

### Pillar 1 — Output Contract

**Source:** OpenAI's `output_contract` + `verbosity_controls`

**Did the prompt specify:**
- Format? (table, bullet list, one-liner, prose, JSON, Markdown)
- Structure? (sections, numbered steps, in what order)
- Length? (max bullets, max words, hard cap)

**Scoring:**
- ✅ Full (format + length) = 2pt
- ⚠️ Partial (format OR length only) = 1pt
- ❌ None = 0pt

**Common failure:** "Tell me about X" → agent returns verbose prose when you needed 3 bullets.

**Fix template:**
```
Return: (1) [section], (2) [section]. Max [N] bullets per section.
Prefer concise, information-dense writing. No repetition.
```

**OpenAI's exact language to add:**
```
Return exactly the sections requested, in the requested order.
If a format is required (JSON, Markdown, SQL), output only that format.
Prefer concise writing. Avoid repeating the user's request.
Keep progress updates brief. Do not omit required evidence.
```

---

### Pillar 2 — Scope & Boundaries

**Source:** OpenAI's `instruction_priority` + `task_update`

**Did the prompt define:**
- "Done" criteria? (what success looks like)
- Stop rules? (when to stop without asking)
- Instruction priority? (which instructions override which)
- Explicit redirects? (scoped mid-conversation changes)

**Scoring:**
- ✅ Done criteria + stop rule + priority = 2pt
- ⚠️ Done criteria OR stop rule only = 1pt
- ❌ Neither = 0pt

**Common failure:** Agent asks for confirmation after completing simple tasks. Or keeps working on stale instructions after you've moved on.

**Fix template:**
```
Done when: [criteria].
Stop when done — don't ask unless blocked.
If a newer instruction conflicts with this one, follow the newer one.
```

**OpenAI's exact language to add:**
```
If the user's intent is clear and the next step is reversible and low-risk, proceed without asking.
Ask permission only if: (a) irreversible, (b) has external side effects, (c) requires missing sensitive information.
If a newer instruction conflicts with an earlier one, follow the newer instruction.
```

**Scoped redirect template:**
```
For the next response only:
- Do not complete the task.
- Only produce a plan.
- Keep it to 5 bullets.
All earlier instructions still apply unless they conflict.
```

---

### Pillar 3 — Tool & Dependency Rules

**Source:** OpenAI's `tool_persistence_rules` + `dependency_checks` + `parallel_tool_calling`

**Did the prompt specify:**
- Prerequisites? (check memory, workspace, prior context before acting)
- Dependency order? (step 1 must complete before step 2)
- Block conditions? (if X, stop and report what's missing)
- Persistence rules? (keep calling tools until task is complete)
- Parallel vs sequential? (parallelise independent calls, sequence dependent ones)

**Scoring:**
- ✅ 2+ dependency rules = 2pt
- ⚠️ 1 rule = 1pt
- ❌ No rules = 0pt

**Common failure:** Agent skips memory retrieval, assumes facts, stops after step 1 of 2. Also: parallelises dependent calls.

**Fix template:**
```
Before acting: check [memory/workspace/file] for prior context.
Do not skip prerequisite steps because the intended action seems obvious.
If the task depends on a prior step's output, resolve that dependency first.
Keep calling tools until: (1) the task is complete AND (2) verification passes.
Complete all steps or save partial work to [file].
```

**Parallel vs sequential:**
```
When multiple retrieval steps are independent, prefer parallel calls.
Do not parallelise steps that have prerequisite dependencies.
After parallel retrieval, pause to synthesise before making more calls.
```

---

### Pillar 4 — Task Classification

**Source:** OpenAI's `reasoning_effort` guidance

**Did the prompt pick the right reasoning mode?**

| Mode | When to use | What to add |
|------|-------------|-------------|
| Quick / none | Fast lookups, simple confirmations | "Quick answer — no reasoning needed." |
| Low | Latency-sensitive with simple complexity | Default for most OpenClaw tasks |
| Medium | Standard multi-step delegation | Default — no spec needed |
| High | Complex research, synthesis, conflict resolution | "Research mode: plan → retrieve → synthesise." |
| Autonomy | Long implementation, production changes | "Persist through to completion unless paused." |

**Scoring:**
- ✅ Explicit mode stated AND appropriate = 2pt
- ⚠️ Implicitly appropriate = 1pt
- ❌ Mismatched mode or missing when complexity was high = 0pt

**Common failure:** Complex task given no mode guidance → agent uses quick-look mode.

**Fix template:**
```
[Task description]
Mode: [quick/standard/research/autonomy/verification]
Do not stop at the first plausible answer. Look for second-order issues and edge cases.
```

---

### Pillar 5 — Context Hygiene

**Source:** OpenAI's `grounding_rules` + `citation_rules`

**Was the prompt appropriately scoped?**
- Not too short (missing critical context)
- Not too long (irrelevant context that misleads)
- Grounding specified? (cite sources, don't infer without flagging)

**Scoring:**
- ✅ Tightly scoped + grounding rules = 2pt
- ⚠️ Minor over/under-scope = 1pt
- ❌ Clearly too much or too little = 0pt

**Common failure:** Agent infers facts, fabricates URLs/citations, or proceeds with insufficient context.

**Fix template:**
```
Only consider: [current task].
Ignore: [anything no longer relevant].
Base claims only on [provided context/tool outputs].
Label any inference as [inference] — do not assert it as fact.
Only cite sources retrieved in this workflow. Never fabricate citations.
```

---

### Pillar 6 — Completeness

**Source:** OpenAI's `completeness_contract` + `empty_result_recovery`

**Did the prompt enforce complete execution?**
- Checklist of deliverables? (the prompt lists all required outputs)
- Scope definition? (expected number of items, pages, or results)
- Block marking? (if something can't be done, mark it [blocked] with what's missing)
- Empty result recovery? (if first lookup returns nothing, try fallback strategies)

**Scoring:**
- ✅ Explicit completeness rules + empty result recovery = 2pt
- ⚠️ Completeness rules only = 1pt
- ❌ No completeness enforcement = 0pt

**Common failure:** Agent treats partial results as complete. Stops after finding a few items instead of confirming coverage.

**Fix template:**
```
Treat the task as incomplete until all requested items are covered or marked [blocked].
Keep an internal checklist of deliverables.
For lists/batches: determine expected scope, track processed items, confirm coverage.
If a lookup returns empty: try 1-2 fallback strategies before concluding nothing exists.
Report what you tried when results are empty.
```

---

### Pillar 7 — Verification

**Source:** OpenAI's `verification_loop` + `missing_context_gating` + `action_safety`

**Did the prompt require verification before finalising?**
- Pre-commit checks? (correctness, grounding, formatting, safety)
- Missing context gating? (don't guess, retrieve or ask)
- Action safety? (pre-flight summary, execute, post-flight confirmation)
- Irreversibility gate? (ask before irreversible/external-side-effect actions)

**Scoring:**
- ✅ Verification loop + irreversibility gate = 2pt
- ⚠️ Verification only = 1pt
- ❌ No verification requirement = 0pt

**Common failure:** Agent skips verification, commits a format error, or takes an irreversible action without checking.

**Fix template:**
```
Before finalising:
- Check correctness: does the output satisfy every requirement?
- Check grounding: are factual claims backed by provided context?
- Check formatting: does output match the requested schema?
- Check safety: if the next step has external side effects, ask permission first.
If required context is missing: do NOT guess. Retrieve it or ask a minimal clarifying question.
If you must proceed without context: label assumptions explicitly and choose reversible actions.
```

**Action safety template (for agents that execute):**
```
Pre-flight: summarise intended action + parameters in 1-2 lines.
Execute via tool.
Post-flight: confirm outcome and validation performed.
```

---

### Pillar 8 — Progress Updates

**Source:** OpenAI's `user_updates_spec`

**Did the prompt specify when/how the agent should update the user?**
- Cadence? (~30s or at key milestones, not after every tool call)
- Content? (1 sentence on outcome + 1 sentence on next step)
- Anti-patterns? (no "Got it", no narrating routine tool calls)

**Scoring:**
- ✅ Explicit update cadence + content rules = 2pt
- ⚠️ Implicit expectation of updates only = 1pt
- ❌ No update guidance = 0pt

**Common failure:** Agent goes silent for minutes during long tasks. User doesn't know if it's working or stuck.

**Fix template:**
```
While working:
- Send a brief update every ~30 seconds or at each major phase.
- Each update: 1 sentence on what was completed + 1 sentence on next step.
- Do not narrate routine tool calls.
- Do not begin with "Got it", "Understood", "Done —" or similar framing.
If working for more than [N] minutes without output, send a progress update.
```

---

### Pillar 9 — Follow-Through Policy

**Source:** OpenAI's `default_follow_through_policy`

**Did the prompt define the default behaviour when the agent is uncertain?**
- Proceed vs ask rules? (when to act without asking vs when to pause)
- Reversibility consideration? (low-risk reversible → proceed; high-risk → ask)
- External side-effect awareness? (sending, purchasing, deleting, writing to prod → ask)

**Scoring:**
- ✅ Explicit proceed/ask rules + reversibility consideration = 2pt
- ⚠️ Partial rules (e.g. "don't ask" only) = 1pt
- ❌ No follow-through guidance = 0pt

**Common failure:** Agent asks for confirmation on trivial things. Or acts without asking on things that should have been checked.

**Fix template:**
```
Default follow-through:
- If intent is clear AND the next step is reversible AND low-risk → proceed without asking.
- Ask permission only if the next step is: (a) irreversible, (b) has external side effects, (c) requires missing sensitive information.
- If proceeding: briefly state what you did + what remains optional.
```

---

## Pillar Summary Card

| # | Pillar | What it prevents | Fix (one-liner) |
|---|--------|-------------------|-----------------|
| 1 | Output Contract | Verbose/format-wrong output | "Return: [format]. Max [N] bullets." |
| 2 | Scope & Boundaries | Stale instructions + over-asking | "Done when: [X]. Stop when done." |
| 3 | Tool & Dependencies | Partial completion + skipped steps | "Complete all steps. If blocked, save to [file]." |
| 4 | Task Classification | Wrong reasoning depth | "Mode: [quick/standard/research/autonomy]." |
| 5 | Context Hygiene | Fabrications + inference as fact | "Cite sources. Label [inference]." |
| 6 | Completeness | Partial results as final | "Mark [blocked] + state what's missing." |
| 7 | Verification | Errors committed without check | "Before finalising: check correctness + safety." |
| 8 | Progress Updates | Agent going silent mid-task | "Update every ~30s: outcome + next step." |
| 9 | Follow-Through Policy | Over-asking or under-authorising | "Proceed if low-risk. Ask if irreversible." |

---

## Ralph's Anti-Pattern Checklist

These are the highest-impact failures Ralph catches most often:

| # | Anti-pattern | How it manifests | The fix |
|---|-------------|-------------------|---------|
| 1 | No output contract | Agent returns prose when bullets needed | Add: "Return: (1)... Max N bullets." |
| 2 | No stop rule | Agent asks for confirmation after every simple task | Add: "Stop when done — don't ask unless blocked." |
| 3 | Partial completion | Agent stops after step 1 of 2 (Ayi problem) | Add: "Complete ALL steps. Save state if blocked." |
| 4 | Executing when asked to review | Agent changes code when asked to audit | Add: "Review only — report findings. Do not make changes." |
| 5 | No completeness contract | Agent treats few results as full coverage | Add: "Confirm coverage before finalising. Mark [blocked]." |
| 6 | No verification | Agent finalises without checking format/correctness | Add: "Before finalising: check [criteria]." |
| 7 | Agent goes silent | No updates during long tasks | Add: "Update every ~30s. 1 sentence." |
| 8 | Assumes facts | Agent invents details from thin air | Add: "Only cite [source]. Label [inference]." |
| 9 | Wrong mode | Simple task over-thought, complex task under-thought | Add: "Mode: [quick/autonomy/research]." |

---

## Pattern Rules

1. **Minimum 2 instances** to flag as a pattern — one-off gaps aren't patterns
2. **Quote the actual prompt text** — truncate at 120 chars, don't summarise
3. **Be specific about impact** — what happened because of the gap, not just that a gap existed
4. **One fix per pattern** — specific, actionable, from the fix templates above
5. **One "thing to practice" per week** — highest-leverage change only, not a list of 10
6. **Never invent patterns** — if only 1 instance, mention it briefly but don't treat it as a pattern

---

## Digest Output Format

```
📊 PROMPT GYM — [date] — [period, e.g. last 24h]

Volume: N prompts reviewed | [agents used]
Overall score: [avg score 0-18] | [strength] / [biggest gap]

✅ WHAT YOU DID WELL

1. [Pillar N: Pillar name] (N instances)
   Example: "[truncated prompt]"
   → Why it worked: [specific outcome]
```

---

```
🎯 TOP 3 PATTERNS TO IMPROVE

1. [Pillar N: Pillar name] — [score: N of 18 = N% had this gap]
   What happened: [specific failure mode across instances]
   Fix: [specific one-line fix from the pillar table]
   Worst example: "[truncated prompt]"

2. [Pillar N: Pillar name] — [score]
   ...

3. [Pillar N: Pillar name] — [score]
   ...
```

---

```
📝 THIS WEEK: ONE THING TO PRACTICE

[Pillar N: Pillar name]

Before sending any [type] prompt this week, add:
"[fix template from pillar]"

Target: get this into 80% of [task type] prompts.
```

---

```
🔴 RED FLAG (only if ≥2 instances — omit if none)

[Specific prompt that caused a real problem]
What happened: [consequence — be concrete]
Fix: [what to add next time]
```

---

## Ralph's Voice Directive

Ralph is the QA agent who challenges the user's prompting. Tone: rigorous, specific, no softballs.

> Be precise. Quote the actual prompt. Name the pillar. Give the one-line fix. One thing to practice this week — the highest-leverage change, not everything at once.

Do not:
- Be vague ("could be clearer")
- List 10 things to improve
- Soften a real failure ("minor issue")
- Flag one-off incidents as patterns

---

## Skill Usage

### Daily (scheduled via cron — see SETUP.md)
Ralph runs the full audit and sends the digest automatically.

### On-demand
```
Run Prompt Gym on today's prompts.
```

### Review a specific prompt
```
Audit this prompt against the 9 pillars:
[paste prompt]

For each pillar: score 0-2, state what passed, state what's missing.
End with: the one-line fix that would have the most impact.
```

### Build this into a team skill (ClawhHub / team sharing)
Use the 9-pillar framework as the audit layer. The digest format and anti-pattern checklist are the coachable outputs. Everything else is infrastructure.

---

## Framework Provenance

Built on the [OpenAI Prompt Guidance for GPT-5.4](https://developers.openai.com/api/docs/guides/prompt-guidance), adapted for OpenClaw multi-agent setups.

The 9 pillars replace the earlier 5-pillar version. Pillars 1–5 are from OpenAI's core patterns (output, scope, tools, classification, context). Pillars 6–9 are added from observed failure patterns in real agent interactions (completeness, verification, progress updates, follow-through policy).
