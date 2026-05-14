---
name: morning-brief
description: Produce the daily operator briefing email from pulse.json + memory + recent events. Runs after scripts/daily_pulse.py. Replaces the legacy exec-assistant agent.
---

# Morning brief

You are producing one email for the operator (oli@tomevault.io). The goal is **a trustworthy daily snapshot, not a feed of agent findings**. If today is genuinely quiet, say so in two sentences and stop. Most days will be quiet.

## Hard rules

1. **The pulse is the source of truth, not the agent events.** Agent events (`pulse.events.items`) are *suggestions* — verify each one against pulse counts and memory before promoting it.
2. **Apply memory hints before drawing conclusions.** `pulse.memory_hints` lists topics likely relevant today. Read each named memory file under `/root/.claude/projects/-home-tomevault/memory/` before writing about a topic the hint flags.
3. **Never promote upstream confidence.** If an event says "boilerplate count may be inflated" → use `roughly_even` or weaker, never `almost_certain`.
4. **No subtraction-based "stuck" claims.** If you're tempted to write "X repos are stuck/unfetched/etc.", check `pulse.pipeline.repos.by_status` first. `gone`, `security_blocked`, `skipped_*`, `cancelled`, `*_error` are terminal — they're not stuck, they're done.
5. **Filter bot traffic from funnels.** If `pulse.traffic.bot_pct > 5`, any PostHog/CTR claim must explicitly note the filter or be omitted. Do not promote unfiltered CTR collapses.
6. **Cross-check "stage dead" claims against `pulse.cron.stages`.** Only flag a stage as broken if `in_cron=true` AND `log_age_min` exceeds its expected interval.
7. **Orphan locks are not crises.** `pulse.cron.orphan_locks` are leftovers from removed stages. Mention once if you've never mentioned them, then drop them.
8. **Classifier health is a state-change-only signal.** `pulse.classifier` carries Haiku triage progress. Surface it ONLY when:
   - `paused: true` (kill-switch fired — surface as RED with the reason)
   - `calls_today.pct_used > 80` (approaching daily ceiling — AMBER)
   - `last_24h.fallback_rate_pct > 10` (Haiku failing → topics-only fallbacks — AMBER)
   - `suspicion_dist_24h` shape shifts unusually (e.g. score≥3 jumps from ~3% baseline to >10% — calibration drift, AMBER)
   - A tier-drain milestone crosses (Tier 1 hits 100% → flag as a green positive, prompts UI flip)
   - `last_classification` is older than 30 minutes (stage stopped silently — RED)
   On a normal day, omit it. The one-line Pipeline summary already captures steady-state.
9. **Verify deploy state before flagging "built but not live" claims.** Before promoting any ticket as "fix shipped but undeployed", check `pulse.deploys.tomevault_web`. If `in_sync: true`, the deploy already shipped — drop the claim. If `in_sync: false` AND `pending_lag_minutes` is small (<30), the deploy is likely in-flight. This addresses the 2026-05-11 false-positive on TOM-536 where a deploy 3.5h prior was flagged as pending.
10. **One operator-facing email. No more than 400 words.**

## Procedure

### 1. Read the pulse
```
cat /home/tomevault/reports/$(date -u +%F)/pulse.json
```
If it's missing, run `venv/bin/python3 -m scripts.daily_pulse` first.

### 2. Read memory index + flagged topics
```
cat /root/.claude/projects/-home-tomevault/memory/MEMORY.md
```
For each item in `pulse.memory_hints`, read the corresponding `*.md` file in the memory dir.

### 3. Read recent events with skepticism
For each `pulse.events.items` entry:
- **Verify the count.** If event says "50,953 stuck", check `pulse.pipeline.repos.by_status`.
- **Apply memory.** If event is about claim CTR and `bot_pct > 5`, the GAE scrape memory likely explains it.
- **Lower confidence on hedged source language.** Auditor saying "may be inflated" → confidence ≤ `roughly_even`.
- **Drop the event** if pulse contradicts it.

### 4. Read yesterday's briefing for continuity
```
cat /home/tomevault/reports/$(date -u -d yesterday +%F)/morning-brief-*.md 2>/dev/null
```
- If yesterday flagged something as RED, **explicitly confirm whether it's resolved** (don't ask the operator to confirm).
- If yesterday flagged the same item, mark it as "still open day N" rather than re-introducing it.

### 5. Decide the briefing mode

- **GREEN** (0 verified red, 0 verified amber after pulse cross-check): one-line email. Subject `TomeVault daily: green`. Body: one sentence + the pulse summary line.
- **AMBER** (1+ verified items): full briefing.
- **RED** (anything pulse-confirmed irreversible or operator-blocking): RED tag in subject.

### 6. Write the briefing

Sections in order:

```markdown
# Daily — YYYY-MM-DD [GREEN | AMBER: N | RED: N]

## TL;DR
<one sentence. include the most decision-relevant fact.>

## Operator decisions (N items)
<only items that need operator action this week. Each:>
- **<title>** — <one sentence what + one sentence why now>
  - Action: <exact command or decision>
  - Memory: <which memory file informed this, if any>

(If none: "No operator decisions today.")

## Pipeline
- Repos: <total> (<active> active, <terminal> terminal)
- Claims: <claimed>/<notified> (<rate>%)
- Distributions: <today> today, <yesterday> yesterday, <total> total
- Drift: <drift> (Meili sync gap)
- New authors: <today>/<yesterday> (baseline 600-800/day)
- Classifier: tier1 <pct>% done, <review_queue> in Sonnet queue, <calls_today>/<ceiling> Haiku calls
  (one line; omit if classifier is paused or off — that goes in the AMBER/RED section instead per Hard Rule 8)

## What changed since yesterday
<diff items with actual numbers from pulse.delta>

## Notes (verified)
<2-4 bullets max. Each must be cross-checked against pulse + memory.>
```

### 7. Save the briefing
```
mkdir -p /home/tomevault/reports/$(date -u +%F)
write to /home/tomevault/reports/$(date -u +%F)/morning-brief-$(date -u +%F).md
```

### 8. Send the email

Use the existing factory email skill:
```python
import sys; sys.path.insert(0, '/home/tomevault')
from scripts.lib.email_sender import send_notification
send_notification(
    to_email="oli@tomevault.io",
    to_name="Oli",
    subject=f"TomeVault daily {date} [{tag}]",
    text_body=briefing_markdown,
)
```

If `TOMEVAULT_ENV != prod`, the call fails closed and writes to `logs/staging-would-have-sent.jsonl` — that's expected during testing.

### 9. Append one line to changelog
```
[morning-brief YYYY-MM-DD] {tag}: N decisions. Verified M events, dropped K.
```

## What to drop entirely from the legacy briefing format

- ❌ "PENDING YOUR DECISION" + "DECISIONS" both as separate sections (collapse into one).
- ❌ "Confidence" labels — operator doesn't act on them. State the action, or don't surface the item.
- ❌ "FACTORY" health section — this is internals; only mention if something is broken.
- ❌ "LOOP HIGHLIGHTS" — vibes, not data.
- ❌ Operator confirmation prompts ("operator should confirm"). The brief should *do* the confirming.

## Verification examples (use as templates)

**Bad (today's briefing):**
> 50,953 repos stuck in discovered (22.9%)

**Good:**
> No items "stuck in discovered" — verified pulse.pipeline.repos.by_status shows 0 in discovered status. The 50k delta from total is `gone` (41k, deleted from GitHub) + `security_blocked` (8k) + skipped (~1.7k). All terminal.

**Bad:**
> Claim funnel CTR collapsed 100x — 23,944 PVs → 14 clicks (0.058%)

**Good:**
> Claim funnel signal is corrupted by GAE bot scrape (bot_pct=14.8% in caddy, per project_gae_claim_scrape). Real human PVs ~14k of 23.9k; CTR is approximately unchanged. Recommend: bot-IP filter on claim_page_viewed event.
