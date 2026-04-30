---
name: session-debrief
description: End-of-session ritual — archive the previous session's notes, write a tight summary of this session's outcomes, update the project state file, enrich the changelog. Bridges sessions so context survives even when the conversation dies. Pattern is universal; specific file paths are project-conventional.
tier: universal
user_invocable: true
---

# Session debrief

Run at the end of a session to capture rich context for the next session. If a session dies before this runs, an auto-capture (e.g. git post-commit hook on the changelog) preserves the bare facts — this skill adds the human layer that makes the next session start with full context instead of cold-starting.

## Conventions this skill assumes

Your project has:
- A "current session" file (e.g. `<project>/.context/sessions/latest.md`, `notes/current-session.md`).
- A "session archive" directory (e.g. `<project>/.context/sessions/archive/`).
- A "project state" file (e.g. `<project>/.context/state.md`) with rolling project status.
- A changelog file (per the `changelog` skill).
- Optionally a "patterns" file (`<project>/.context/patterns.md`) for gotchas + conventions.

These names are by convention; substitute your project's equivalents. The pattern is the same.

## Behaviour

### 1. Archive the previous session summary

If the "current session" file exists:
- Copy it to the archive directory under a name keyed to its session date — *not* today's date. The session file's own date heading is the canonical date.
- If the archive grows beyond a threshold (typical: 10 individual files), compress the oldest into a quarterly digest (one paragraph per session: objective + outcome + anything still load-bearing).

### 2. Write the new session summary

Use this template in the "current session" file:

```markdown
# Session: {YYYY-MM-DD} — {short tagline}

> **Picking up next session?** {one paragraph: what's open, what's stable, what to read first}

## Operator objective

{What the operator asked to accomplish this session, in their voice as far as possible.}

## What shipped

{Bullet list of what got done. Group by repo or domain if the session spanned multiple. Include commit SHAs.}

## What didn't ship (and why)

{Anything started but not completed, scope-pulled, or deferred. Reason is more useful than detail.}

## Decisions made

{Any decisions with brief rationale. Cross-link to a decision memo if one exists.}

## Blocked on

{Anything waiting on operator action, external service, unresolved question.}

## What did NOT change

{Things that stayed untouched on purpose — config files, secrets, system services.
This is more important than it looks: future-you will spot if any of these change.}

## Next session default pickup

{What logically comes next based on current state. One concrete first move.}
```

Keep the file ~30-60 lines max. The "Picking up next session?" header at the top is load-bearing — it must answer "if I open this cold, what do I need to know?" in one paragraph.

### 3. Update the project state file

Overwrite the state file with:
- Git branch and last commit (from `git log -1 --oneline`).
- Active focus area (1-2 sentences).
- Current blockers (3-5 bullets max).
- Infrastructure status (which services are healthy / degraded / off).
- Any kill-switches active.

The state file is what an agent reads at *session start*. The session file is what an agent reads at *session pickup*. Both should answer the same question — "where are we?" — at different fidelity.

### 4. Enrich the changelog

Review today's auto-captured changelog entries. For any significant items, add a `**[category]**` enriched entry with the "why" context using the `changelog` skill.

Aim for 1-3 rich entries per session, not one per commit. Most commits don't need a rich entry; a few do.

### 5. Update patterns

If any new gotchas, conventions, or non-obvious facts were discovered during the session, append them to the patterns file. Examples:

- "Plugin manifest updates require a daemon restart, not just a reinstall."
- "The audit script reads from the live weights file, so default-template changes need both files updated."
- "This vendor's webhook payload differs between docs and reality — they actually send X, not Y."

Patterns survive across sessions and team members; the changelog and state file decay faster.

### 6. Remind about commit + push

After all context files are updated, surface the commit instruction:

```
git add <context-dir>/ && git commit -m "docs: session debrief {YYYY-MM-DD}" && git push
```

Replace `<context-dir>` with the project's actual context directory. The reason this is in the skill: half of the value of a debrief is lost if it doesn't survive a machine reboot. Push.

## Calibration signals

Healthy debrief discipline:
- Next session opens, the agent reads the latest session file, and the operator does not need to re-explain the state.
- Decisions made in session N are still findable in session N+5 because the rich changelog entry is there.
- The patterns file gets a steady ~1 entry per session and rarely shrinks.

Unhealthy debrief discipline:
- Session files all start identically with "operator opened a chat" — the template is being filled mechanically rather than reflecting the actual session.
- The patterns file is empty or hasn't grown in a month.
- Cold-starting an agent requires the operator to re-explain state.

## Anti-patterns to refuse

- **Replaying the session.** The session file is for the next session, not for archival completeness. Compress.
- **Hiding what didn't ship.** "What didn't ship" is more diagnostic than "what shipped." Be honest.
- **Empty `Decisions made` sections.** Most sessions contain at least one decision. If the section is empty, you missed it.
- **Updating the patterns file with code-level conventions.** Those go in the codebase. Patterns is for non-obvious *operational* facts.

## Pairs with

- `changelog` — debrief invokes changelog for the rich entries.
- `decision-memo` — decisions surfaced in the debrief should reference their memo, not duplicate it.
- `memory-write` — durable facts (operator preferences, recurring incident patterns) move into memory; transient session state stays in the debrief.
