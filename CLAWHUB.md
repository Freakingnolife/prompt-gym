# Publishing on ClawhHub

ClawhHub (https://clawhub.com) is OpenClaw's skill marketplace. This guide covers how to package and publish Prompt Digest there.

## Package Structure for ClawhHub

ClawhHub skills are distributed as a single `SKILL.md` with embedded documentation. The `prompt-digest` skill is self-contained — the `SKILL.md` already has everything needed:

```
prompt-digest/
└── SKILL.md  ← The only file ClawhHub needs
```

The SKILL.md already has:
- Frontmatter with name, description
- Full 5-pillar audit framework
- Digest output format
- Ralph's voice directive
- Anti-patterns to catch
- Configuration reference

## Publishing Steps

### 1. Clean the SKILL.md

Before publishing, remove any Marcus-specific config references. The skill should be generic:

- [ ] Remove `MARCUS_DISCORD_ID` references
- [ ] Remove `DM_CHANNEL_ID` references  
- [ ] Remove any hardcoded paths specific to Marcus's setup
- [ ] Update configuration section to use placeholder examples

### 2. Test the Skill

Before publishing, verify it works in isolation:

```bash
# In your OpenClaw session:
/skill prompt-review
# Should load the 5-pillar framework
```

### 3. Create ClawhHub Account

1. Go to https://clawhub.com
2. Connect your OpenClaw account
3. Navigate to "Publish a Skill"

### 4. Submit

- **Skill name:** `prompt-digest`
- **Tagline:** "Ralph reviews your prompts against the OpenAI guidance framework — daily, automatically."
- **Description:** Copy from README.md summary
- **Category:** `productivity` or `coaching`
- **Files:** Upload `SKILL.md` only
- **Price:** Free (recommended to start)

### 5. Metadata

```yaml
name: prompt-digest
description: Ralph's AI prompt coaching via daily digest. Review your prompts 
            against the OpenAI prompt guidance framework, delivered to Discord daily.
category: productivity
tags: [prompting, coaching, daily-digest, openai, discord, ralph, qa]
author: your-github-handle
version: 1.0.0
compatibility: ">= 1.0.0"
```

---

## Alternative: Share as OpenClaw Workspace Package

If you want to share the full system (skill + cron + templates):

```bash
# Package the entire repo
zip -r prompt-digest.zip prompt-digest/ \
  --exclude "prompt-digest/.git/*" \
  --exclude "prompt-digest/CLAWHUB.md"

# Upload to GitHub Releases
gh release create v1.0.0 \
  --title "Prompt Digest v1.0.0" \
  --notes "Ralph's daily prompt coaching skill. Full system: skill + cron + templates."
```

---

## What ClawhHub Does and Doesn't Do

| ClawhHub handles | ClawhHub does NOT handle |
|-----------------|------------------------|
| Skill distribution | Cron job setup (manual step) |
| Skill discovery | Discord bot configuration |
| Skill installation | Ralph's Discord account |
| Skill updates | Environment variables |

Users will still need to:
1. Install the skill (`cp SKILL.md ~/.openclaw/skills/prompt-review/SKILL.md`)
2. Set up the cron job (follow CRON_SETUP.md)
3. Configure Discord (follow CONFIG.md)

---

## Versioning

| Version | When | What changed |
|---------|------|-------------|
| 1.0.0 | Initial | Full system: 5-pillar framework + digest + cron |
| 1.1.0 | (Future) | Auto-setup script, CLI config wizard |
| 1.2.0 | (Future) | Multi-user support, team digest |

---

## Promoting on ClawhHub

- Add a demo GIF of the digest arriving in Discord
- Write a 1-paragraph use case in the description
- Cross-post to:
  - OpenClaw Discord (#showcase channel)
  - X/Twitter with #OpenClaw #AIautomation
  - LinkedIn (AI automation / productivity niche)
