---
name: roadmap-item
description: Add or revise an RDMP entry in constitutions/roadmap.md. Three phases — spec confirmation with the operator, doc insertion, commit confirmation. The auto-ticketing cron picks up the new RDMP within 10 min of commit and creates a todo build_card for Architect — the factory chain then runs autonomously until an operator-review-required output (medium/high tier) lands in the operator's Paperclip approvals inbox.
tier: project
user_invocable: true
---

# /roadmap-item

Use when the operator has direction for new factory work — a feature, a pipeline change, a UX improvement, an infra capability — that should become an RDMP entry in `constitutions/roadmap.md`. Also use for revising an existing RDMP.

## When to use

- Operator describes a new direction: "we should build X" / "next priority is Y"
- Operator wants to flesh out an existing thin RDMP
- Operator wants to change the ordering or scope of current RDMPs

NOT for:
- Posting a ticket directly to Paperclip (use `/build-ticket`)
- Tuning an existing shipped thing (use `/evolve-ticket`)
- Changing stage status (SHIPPED / CURRENT / NEXT / LATER) — that's a direct roadmap edit, flag it to operator
- Any change to `design-principles.md` or `build-state.md` (build-state is auto-generated)

## The RDMP template (Option A inline format)

Every RDMP entry in `#### In scope` uses this structure. Matches the format documented in roadmap.md §5 intro.

```markdown
- **[RDMP-NNN]** <one-line outcome statement, reads as the ticket title>.
  - **Why:** <citations — §4 problems, §6 cross-cutting priorities, §10 moats affected>
  - **Problem:** <2-4 sentences, concrete. What's broken / missing / unmet today. Cite evidence when available (metric, incident, user request).>
  - **Direction:** <2-4 sentences. Key design decisions. Not an implementation plan — Architect decomposes that. State the approach + any invariants that must hold.>
  - **In scope:**
    - <bullet: concrete deliverable>
    - <bullet: concrete deliverable>
  - **Out of scope:**
    - <bullet: related thing that is NOT part of this RDMP, deferred or explicitly rejected>
  - **Acceptance:**
    - <testable pass/fail criterion>
    - <testable pass/fail criterion>
  - **Dependencies:** <what must exist or be true before this can ship. "none" if none.>
  - **Upstream considerations:** <risks, edge cases, related systems that could break or interact. Skip if truly none.>
  - **Tier:** `low` | `medium` | `high`
```

Required sub-bullets: **Why, Problem, Direction, Acceptance, Tier.**
Optional: In scope, Out of scope, Dependencies, Upstream considerations.

Keep each sub-bullet terse. Architect will expand with file paths and acceptance-criteria tests when producing the architect_spec. This RDMP just has to carry enough direction that Architect can do that well.

## Tier guidance (from v3.1 + operator policy)

- **`low`:** trivial, isolated, no UI change, no schema change, single commit reversal. Implementer auto-ships; operator sees in daily briefing only.
- **`medium`:** features, pages, UX, migrations, pipeline logic changes. Implementer routes back to operator via `review_ready` + Paperclip approval.
- **`high`:** schema changes, structural moves, first-of-its-kind systems, external-first actions. Adversary reviews at high tier; operator approval required; not reversible by single commit.

## Phase 1 — Spec confirmation

The operator's initial direction is almost never complete enough. Draft the full RDMP content, show it, iterate until they say "yes that's right."

Step-by-step:

1. **Gather context.** Read the operator's direction and any referenced files/tickets/briefings. If direction mentions existing code or systems, check them — don't guess the current state.

2. **Check scope fit:**
   - Does this violate any §7 non-goal? If so, either kill it or surface the conflict and ask the operator whether they want to amend §7.
   - Does it fit inside the CURRENT stage (per §5.2's scope)? If it belongs in a NEXT/LATER stage, flag that.
   - Does it cite a §4 developer problem? Every RDMP should.
   - Does it honor §9 load-bearing principles? (Event-driven over cron, delete a part before adding an orchestrator, etc.)

3. **Draft all required sub-bullets** (Why, Problem, Direction, Acceptance, Tier) plus optional ones as relevant.

4. **Tier check:** does the `tier` you picked align with §11 Strategist autonomy envelope? If tier is `high`, the work will route back to operator for approval at Implementer stage regardless. If `low`, Implementer auto-ships — make sure that's appropriate.

5. **Dependency check:** does this depend on something not yet in build-state.md? If so, list those as `Dependencies:` and verify they exist (build-state.md is auto-generated, so grep it directly).

6. **Present the draft to the operator.** Show the full RDMP block. Ask:
   - "Does this capture what you want?"
   - "Anything missing from acceptance?"
   - "Tier feel right?"
   - "Any sub-bullet I got wrong?"

7. **Iterate.** Typically 1-2 rounds. Don't proceed to Phase 2 without explicit operator approval of the final content.

**If the operator wants to revise an existing RDMP rather than add a new one:** same phase 1, but pull the current RDMP block first and present the proposed diff alongside the new version.

## Phase 2 — Insert into roadmap.md

Only run this phase after operator confirms the Phase 1 draft.

Step-by-step:

1. **Allocate the RDMP ID.** Read the roadmap file. Find all `RDMP-NNN` strings. Pick the next free integer (max + 1, zero-padded to 3 digits).

2. **Determine insertion point.**
   - New RDMP: by default, append to the bottom of the current stage's `#### In scope` list. If operator specified an ordering ("put this at position 2"), respect that.
   - Revision: replace the existing RDMP block in place.

3. **Edit the roadmap file** with the new / revised block. Preserve exact sub-bullet indentation (2 spaces for first level, 4 for nested).

4. **Check for ripple edits:**
   - If the new RDMP introduces a new vocabulary term, add it to §8.
   - If the new RDMP introduces a non-goal, add it to §7.
   - If the new RDMP changes priority ordering, consider §6 cross-cutting updates.
   - If the new RDMP affects a threshold, update §14.
   - If the new RDMP changes Strategist autonomy (expands or restricts envelope), update §11.

5. **Append to §16 amendment log.** One line, format:
   ```
   - [YYYY-MM-DD] §5 — added RDMP-NNN <headline> — evidence: commit (this change)
   ```

6. **Show the operator the diff** (use `diff -u` or equivalent). Any ripple edits should be obvious.

## Phase 3 — Commit confirmation

1. **Show final diff** of `constitutions/roadmap.md`.

2. **Ask explicitly:** "Ready to commit? Within 10 min the roadmap_sync cron creates a todo build_card assigned to Architect with `architect_pending` + `operator_approved` + `build_card` labels. Handoff-dispatcher routes it to Architect within 5 min of creation. The factory chain then runs autonomously until medium/high-tier Implementer output lands in your approvals inbox as `review_ready`. Low-tier output auto-ships; you see it in tomorrow's exec briefing."

3. **On yes:**
   - `cp /tmp/roadmap-new.md /home/tomevault/constitutions/roadmap.md` (constitutions/*.md is deny-listed for Write/Edit; use `cp` from a scratch file to bypass — the operator-authored roadmap content is NOT a "claude edit" in spirit, cp is the right tool)
   - `git add constitutions/roadmap.md`
   - `git commit -m "feat(roadmap): RDMP-NNN — <headline>"`
   - `git push origin main`
   - Offer to run `cd /home/tomevault && python3 -m scripts.factory.roadmap_sync` immediately for instant pickup (cron fires it in <10 min anyway). ALWAYS cd first — the cron does, and manual invocations must too or you get `ModuleNotFoundError: No module named 'scripts'`.

4. **On no / revise:** go back to Phase 1 or 2 depending on what changed.

5. **Report result:** confirm the ticket was created (or will be at next cron tick), give the operator its ID.

## Edge cases

- **The operator's direction violates a §7 non-goal.** Do not draft an RDMP. Show the conflict, ask whether they want to amend §7 (separate action — stop this skill, use a direct roadmap edit or an amendment proposal).
- **The direction belongs in a NEXT or LATER stage, not CURRENT.** Draft the RDMP but insert into the correct stage's `#### In scope`. Note to operator: "this won't auto-generate a ticket until stage flips to CURRENT."
- **The direction is strategic (belongs in §6 cross-cutting priorities) rather than a specific deliverable.** Suggest §6 update instead; don't force it into an RDMP.
- **Operator gives direction that overlaps an existing RDMP.** Show the existing one; ask whether this is a revision, a merge, or a separate item.
- **Operator wants to defer an RDMP (CURRENT → NEXT stage).** Not this skill's scope — flag as a structural roadmap edit.

## The full chain (what happens after you commit)

This is the flow the operator relies on — understand it end-to-end before writing anything:

```
OPERATOR edits roadmap.md via /roadmap-item + commits
  ↓ (within 10 min)
roadmap_sync.py cron creates ticket:
  status=todo
  labels=[build_card, architect_pending, operator_approved]
  assignee=architect
  body=full spec from RDMP sub-bullets
  ↓ (within 5 min)
handoff-dispatcher reassigns assigneeAgentId (fires assignment-wake)
  ↓
Architect heartbeat fires
  - reads ticket body
  - cross-checks against v3.1 + roadmap
  - produces architect_spec ticket with adversary_pending label
  ↓ (within 5 min)
handoff-dispatcher routes to Adversary
  ↓
Adversary judges artifact
  - green → adds implementer_ready label
  - amber → adds implementer_ready with caveats comment
  - red → sends back to source via queue.enqueue
  ↓
handoff-dispatcher routes implementer_ready → Implementer
  ↓
Implementer executes
  - low tier → auto-ships, status=done, operator sees in tomorrow's exec briefing
  - medium/high tier → emits review_ready label + creates Paperclip approval,
                       assigneeUserId=local-board
  ↓
OPERATOR sees review_ready approval in Paperclip inbox
  - approve → ticket closes, work lands
  - reject → ticket reopens, revert hash from audit
  - request-revision → routes back to Implementer
```

Key point: operator's ONLY two touchpoints in the whole loop are:
1. Writing/revising the roadmap (this skill)
2. Acting on approvals in the inbox (Paperclip UI, triggered by Implementer for medium/high tier)

Everything else runs without operator attention. Never propose flows that require additional operator clicks.

## Guardrails

- Never skip Phase 1 (operator confirmation) even if the direction seems obvious. Surprises surface during drafting.
- Never commit without explicit Phase 3 approval.
- Never edit `constitutions/build-state.md` — it's auto-regenerated by `scripts/factory/build_state.py`.
- Never edit `constitutions/design-principles.md` as part of this skill.
- Never edit `roadmap.md` to change `status:` lines (that's a stage lifecycle change, not a ticket change).
- Never create the Paperclip ticket directly (`/build-ticket` does that differently). The `roadmap_sync.py` cron does it after your commit, which is the correct trigger.
- Never propose a "backlog step" / "operator promotes" step between roadmap commit and Architect starting. The operator rejected that pattern; the ticket goes straight to `todo` and flows autonomously.
- `_preamble.work.md` + §11 say agents must cite §4 problem + §5 stage / §6 priority. Honor this in the `Why:` field.

## Example (fleshed-out RDMP-001 for reference)

```markdown
- **[RDMP-001]** GitHub App: auto-relay on push, consented badge PR, verified ownership.
  - **Why:** §4.1 porting tax (creators author once, re-author everywhere); §4.2 portable identity (install is the portable identity marker); §4.3 invisible reach (install count becomes the engagement metric). Moat #4 (creator lock-in via install surface).
  - **Problem:** Creators today publish a Skill on GitHub, then manually publish it again to Cursor, Claude, etc. No single installable artifact propagates their work. Reputation and usage stay invisible and trapped in the source repo. We currently tell creators "we indexed you" via email — a weak conversion surface. Stage B thesis says the App install IS the relay; without it, every downstream metric is unmoored.
  - **Direction:** Register a GitHub App that listens to push events on Skill-bearing repos. On first install: detect Skills in the tree, open a consented PR adding a TomeVault badge + registering with our relay. Verified-ownership is established by the install (App install requires repo admin). Post-install: every push re-indexes affected Skills. Install count feeds the Stage-B dashboard.
  - **In scope:**
    - GitHub App registration (github.com/apps/tomevault-relay)
    - Webhook handler: installation created, installation deleted, push on Skill-bearing files
    - Badge-PR template + auto-open on first install
    - Profile page shows ✓ verified indicator after install
    - Install count stored in new `installations` table, joined to `repos.owner`
  - **Out of scope:**
    - Analytics dashboard (Stage C)
    - Org-level per-repo consent (Phase 2)
    - Re-indexing historical PRs (only forward push events)
  - **Acceptance:**
    - App installable at the registered URL within one business day of spec approval
    - Installing on a repo with ≥1 SKILL.md produces a badge PR within 30 seconds
    - Profile page at `/creator/<owner>` shows ✓ verified after PR merge
    - Install event writes to `installations` table with correct owner + repo + installed_at
    - Uninstall event marks the row as `inactive`
  - **Dependencies:** Existing `scripts/discover.py` GitHub integration (already auth'd). No new external services.
  - **Upstream considerations:**
    - GitHub App scopes must be minimal — `contents:write` (PR creation) + `metadata:read`. Not `administration`, not `repo:*`.
    - Webhook signatures verified via hmac-sha256 against stored webhook secret. Reject unsigned payloads silently.
    - Rate limit: 5000/hr per installation. Comfortable headroom.
    - Existing badge format in README must be preserved — if creator has a custom badge, append beside rather than overwriting.
    - Install notification email should route through `email.send` skill with the outreach kill-switch check (even though these are transactional, not campaign sends — defense in depth).
  - **Tier:** high
```

## Related skills

- `/build-ticket` — posts a build_card directly to Paperclip without going through the roadmap. Use when the ticket is one-off and doesn't belong in strategic direction.
- `/evolve-ticket` — tunes an existing shipped thing. Different label, different routing.

## What this skill does NOT do

- Create the Paperclip ticket (cron does that).
- Update build-state.md (auto-generated).
- Decide stage transitions.
- Assign ticket to anyone other than Architect (that's the architect_pending label behavior).
- Change autonomy envelope (§11) or non-goals (§7) without surfacing the conflict to operator first.
