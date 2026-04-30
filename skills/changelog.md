---
name: changelog
description: Add a rich context entry to the project's rolling changelog beyond what an auto-hook captures. Use after a significant feature, fix, refactor, or decision — the why beyond the commit message.
tier: universal
user_invocable: true
---

# Changelog

Add a human-curated entry to your project's changelog with richer context than the auto-captured commit messages provide. The auto-capture is for completeness; the curated entry is for the future reader who needs to understand *why*, not just what.

## When to use

- After completing a significant feature, fix, or refactor — to add the **why** beyond the commit message.
- To log something that wasn't a commit: a decision, a discovery, a pattern learned, a vendor change.
- NOT for trivial edits, typo fixes, or formatting changes (those auto-capture is enough).

## Conventions this skill assumes

Your project has a changelog file at a known location (e.g. `<project>/CHANGELOG.md`, `.context/changelog.md`, `docs/changelog.md`). It is rolling — entries grouped by date, newest first. It auto-captures commit messages via a post-commit hook (or similar), and the rich entries live alongside the auto-captured lines.

If your project doesn't have this convention yet, set it up first — the rolling file plus the auto-capture hook is the load-bearing infrastructure; the rich entries are the layer on top.

## Arguments

When invoking, provide:
- **category**: One of `feature`, `fix`, `refactor`, `infrastructure`, `docs`, `decision`.
- **summary**: One sentence explaining what was done and **why**.

## Behaviour

1. Read the project's changelog file.
2. Find or create today's date heading (`## YYYY-MM-DD`).
3. Append: `- **[{category}]** {summary}`
4. If the file exceeds the project's size threshold (typical: 100 lines), move the oldest date-group to a quarterly archive (e.g. `changelog-archive/{YYYY}-Q{N}.md`).

## Format

```markdown
## 2026-04-05

- feat: classify content_type at sync time (auto-captured)
- **[feature]** Added content_type classification so the frontend can filter content by type. Driven by the new taxonomy decision.
- fix: handle empty result set in pagination (auto-captured)
- **[decision]** Picked client-side pagination over server-side because the search index already returns full result sets and the page sizes never exceed 5K.
```

Rich entries (with `**[category]**`) sit alongside auto-captured commit messages (plain `- feat:` lines). This makes it easy to scan: bold prefix = curated context, plain prefix = automated record.

## What to write in the summary

Good summary:
> **[feature]** Switched search index to use array fields for tags rather than comma-joined strings. Avoids the multi-select-filter bug we couldn't reproduce with the old shape.

Bad summary (just restates the commit):
> **[feature]** Added array fields for tags.

The good summary names the *why* (multi-select bug), the bad one duplicates the commit message. If the rich entry only restates the commit, don't add it; the commit is enough.

## When to write `[decision]` entries

Decisions that don't have a commit but matter:
- Vendor change ("decided to migrate from X to Y").
- Architectural choice with no immediate code change ("staging environment deferred until DAU > N").
- Operator preference recorded ("we don't want feature X for these reasons").

These are the most valuable changelog entries because they wouldn't survive otherwise — there's no commit message to capture them.

## Anti-patterns to refuse

- **Restating commits.** If the commit message is good, no rich entry needed.
- **Bulk dumps.** "Today we did 12 things." Pick the 1-2 that matter; the rest is auto-capture.
- **Self-praise.** Changelog is for the next reader, not the author. Keep tone neutral.
- **Forward-looking entries.** "Going to ship X next week." That's a roadmap or session note, not a changelog.

## Pairs with

- `session-debrief` — at session end, the debrief enriches the changelog with the day's `[decision]` entries.
- `decision-memo` — when a decision was made via memo, the changelog captures the headline; the memo is the appendix.
