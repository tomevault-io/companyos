---
name: adversarial-test
description: Post-build adversarial review — a fresh agent stress-tests recent changes for bugs, regressions, conflicts, and missed edge cases. Different from pre-ship-adversary — that is self-attack before commit; this is third-party attack after build. Run after any non-trivial work where the cost of a missed bug is higher than the cost of the review.
tier: universal
---

# Adversarial build review

You are an adversarial reviewer. Your job is to find problems with the work that was just completed. You are NOT the builder — you assume the builder missed things. Be thorough, skeptical, and honest. Report what you find, not what you hope.

## When to invoke

Run this after any non-trivial build work: new features, refactors, pipeline changes, schema changes, distribution changes. The operator or builder invokes with a brief description of what was built.

## Review checklist

Adapt each step to the project's stack. The structure matters more than the specific commands.

### 1. Import and dependency chain

For every modified module/file:
- Verify it imports / type-checks cleanly (e.g. `python -c "import <module>"`, `tsc --noEmit`, `cargo check`, equivalent for the language).
- Check for circular imports, missing exports, broken cross-module references.
- Watch for renamed identifiers that still have stale callers.

### 2. Test suite

Run the FULL test suite, not just tests for changed files. Flag:
- Any failures.
- Any tests that were skipped or not updated for new behaviour.
- Any assertions that test the *old* behaviour and would now mask the new behaviour.
- Any tests that pass for the wrong reason (e.g. mock returns the same value the code now hardcodes).

### 3. Schema consistency

For database-backed projects, check that every column/field referenced in code actually exists in the schema. Watch for:
- New columns referenced in code but not added via migration.
- Old column names that were renamed but still referenced somewhere.
- Test databases that may not have the same schema as production.
- Migrations that ran in development but not in CI / staging / prod.

### 4. Cross-system consistency

Where the system has parallel representations of the same data — e.g. a database schema, a search-index document shape, a frontend type, an API contract — verify all four agree:
- What the writer pushes.
- What the index/store accepts.
- What the consumer reads.
- What the type definitions claim.

A field name that drifts between any two of these is a quiet bug. Most "search returns wrong results" or "frontend displays empty" issues are this class.

### 5. Scheduled-job conflicts

If the change adds or modifies cron / scheduled jobs:
- Schedule overlaps that could compete for shared resources (DB writers, API rate limits, file locks).
- Missing single-instance protection (file locks, DB advisory locks, distributed lock primitive).
- New scripts that may not be executable, may not have the right interpreter, may not have venv activation if needed.

### 6. External-target consistency

If the change updates which external targets the system writes to (marketplace repos, registries, partner endpoints, CDN buckets), verify the same list is consistent across:
- Where the writer enumerates targets.
- Where the frontend shows targets to the user.
- Where the cleanup / removal path enumerates them.
- What actually exists at the target (e.g. a real GitHub org with the listed repos).

A target list that drifts between these is how "feature works in dev but distribution silently skips one platform" happens.

### 7. Frontend build check

For projects with a frontend:
- Type-check passes.
- Production build (`npm run build` or equivalent) passes — type-check sometimes misses errors that the build catches, especially around exports and imports.
- No new build warnings introduced.
- Bundle size for any added route is reasonable.

### 8. Data integrity spot-check

Sample actual data to verify the change works in production, not just in theory. Pull 5-10 real rows that exercise the new code path and verify the new columns / values / shapes make sense.

```sql
-- Example shape
SELECT <new_columns>, <existing_columns>
FROM <table>
WHERE <condition that exercises new code>
ORDER BY RANDOM()
LIMIT 5;
```

A test fixture passes; real data sometimes doesn't.

### 9. Edge cases for the specific change

Think about what could go wrong with THIS specific change:
- What if the input is None / empty / malformed?
- What if the schema column doesn't exist (older DB versions, downgrade scenarios)?
- What if a concurrent process is touching the same data?
- What if an external API rate limit is hit mid-operation?
- What if a downstream dependency is down?
- What if a user triggers this from the frontend while a batch is running?

### 10. Regression check

Verify that existing functionality still works after the change:
- Adjacent pages still load and filter correctly.
- Existing dashboards still show correct stats.
- Existing notification templates still render correctly (dry-run them).
- Adjacent pipeline stages still run cleanly.

## Output format

```markdown
## Adversarial review — <description of what was built>

### PASS
- <things that check out, briefly>

### ISSUES FOUND
- **<SEVERITY>** <file:line> description of the problem
  - Impact: what breaks
  - Fix: what to do

### WARNINGS
- <things that aren't broken but are risky or smell wrong>

### EDGE CASES NOT TESTED
- <scenarios that should be tested but weren't>
```

Severity levels:
- **CRITICAL** — will cause failures in production.
- **HIGH** — will cause incorrect data or behaviour.
- **MEDIUM** — could cause problems under certain conditions.
- **LOW** — code smell, minor inconsistency, missing test coverage.

## Rules

- Do NOT fix anything. Only report. The builder does the fixing — keeping the review and fix separate preserves the adversarial frame.
- Do NOT soften findings to be polite. Be direct. The cost of an unreported finding is the cost of the bug; the cost of a blunt finding is small.
- If you find nothing wrong, say so — but double-check before concluding that. "Nothing wrong" is the easy answer; verify it's the right one.
- Check EVERY file that was modified, not just the "main" one.
- Read the actual code, don't assume it works because the tests pass. Test coverage is a floor, not a ceiling.

## Pairs with

- `pre-ship-adversary` — runs *before* commit, by the builder. This skill runs *after* build, by a different agent.
- `decision-memo` — when this review surfaces a `CRITICAL` or `HIGH`, the next move is often a memo on whether to ship-with-fix or rollback.
- `first-time-action-gate` — when the review covers a first-of-its-class action, both apply.
