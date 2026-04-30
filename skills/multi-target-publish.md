---
name: multi-target-publish
description: When a single change spans multiple repos / packages / deploy targets with different deploy mechanisms (auto-deploy poll vs manual push vs CDN invalidation vs npm publish), the order matters and each target needs its own verification. This skill is the sequencing pattern: pre-checks, deploy-order, per-target verification, rollback awareness. Cheaper than a per-project bespoke runbook for every release.
tier: universal
---

# Multi-target publish

Most non-trivial changes ship across more than one target: a frontend repo and a backend repo; an npm package and a documentation site; a database migration and the application code that uses it. Each target has its own deploy mechanism, its own verification probe, and its own rollback shape. Treating "the deploy" as a single atomic action is how partial-deploy bugs land in production.

This skill is the universal sequencing pattern. It works whether your targets are git repos, npm packages, Docker images, S3 buckets, Cloudflare Workers, or any combination.

## When to apply

A change is multi-target if any of these hold:
- It touches code in more than one repo or package.
- It introduces a schema change that exists in one place (migration file) and is *consumed* by code in another place.
- It requires a coordinated rollout across different deploy mechanisms (auto-poll vs manual vs CDN purge vs registry publish).
- It depends on a configuration change that's deployed separately from the code change (env var, feature flag, secret rotation).

Don't apply when:
- The change is purely additive in a single target (one repo, one deploy).
- The change is a config-only flip with no code change.
- The targets are decoupled and the order genuinely doesn't matter (rare; usually at least one ordering constraint exists).

## The sequence

### Step 0 — Map the targets

Before anything else, write down every target the change touches. Common shapes:

```
Targets:
1. <repo A> — <deploy mechanism, e.g. "auto-deploy poll on origin/main">
2. <repo B> — <deploy mechanism, e.g. "manual git push, no auto-deploy">
3. <npm package> — <"npm publish, version bump required">
4. <database> — <"migration applies on next service restart">
5. <CDN cache> — <"purge required after frontend deploy">
```

For each target, note:
- The deploy *trigger* (what action causes deploy to start).
- The deploy *latency* (how long from trigger to live).
- The deploy *visibility* (how do you know it's done?).
- The *rollback* shape (revert commit? unpublish? re-deploy old version?).

If the map has more than one target you're not sure about, pause and ask the operator. A wrong deploy mechanism guess is more expensive than the question.

### Step 1 — Pre-checks per target

Before any push, every target must pass its own pre-check:

```
For each target:
- Build / type-check / lint passes locally.
- Test suite (or smoke test) passes.
- The change is staged but not yet committed.
- No unrelated changes are accidentally bundled.
```

The hardest part is "no unrelated changes." Multi-target work tends to brush against in-flight changes from other sessions. Use `git diff --stat` per target before staging; verify each modified file is in scope.

### Step 2 — Sequence by dependency, not convenience

Targets must deploy in the order their dependencies require. Common ordering constraints:

- **Database migration before code that uses the new schema.** If the code deploys first, it 500s on every request to the new field until the migration lands.
- **Backend API before frontend that calls the new API.** Same shape — frontend can't call something not yet deployed.
- **Schema-additive migrations before code; schema-removing migrations after code.** Removing a column the code still reads is the most common ordering bug.
- **Package publish before code that depends on the new version.** If the consumer ships first with `^1.1.0` in its dependency list, install fails until the package is on the registry.
- **Auto-deploying targets after manual targets** when both are needed at the same logical moment. The auto-deploy will fire on its next poll; the manual one needs you to push it. Ship the manual one first so that when the auto-deploy lands, the manual one is already there.

If you can't determine ordering from dependencies alone, default to:
1. Schema changes first (always backward-compatible additions).
2. Backend / API changes second.
3. Frontend / consumer changes third.
4. Package publishes last (so the registry artefact is the public commitment).

### Step 3 — Verify per target before continuing

After each target deploys, verify before moving to the next. The verification should be observable:

```
For target N:
1. Trigger fired? (commit pushed, npm publish ran, migration applied)
2. Deploy latency elapsed?
3. Live state reflects the change? (probe an endpoint, query a row, check a header)
4. No regressions visible? (existing tests still green; existing endpoints still 200)
```

Don't fire all targets in parallel and verify at the end. If target 2 broke something, target 3 may compound the failure or block your rollback.

### Step 4 — Capture the deploy as one atom

After all targets have deployed and verified, capture the rollout as a single record:

- Commit message in each target references the others ("see <other repo> commit <sha> for the matching change").
- A summary in your project's changelog or session log: "Shipped <change> across <N> targets at <time>."
- If the change is risky enough to warrant rollback awareness, write down the rollback steps in the same record so they're ready if needed.

### Step 5 — Be ready to roll back asymmetrically

Multi-target rollback is harder than single-target because each target has its own rollback shape:

- **Git repos:** revert commit, push.
- **npm packages:** unpublish (within 72h) or publish a patch.
- **Database migrations:** down-migration if reversible; data-migration script if not; in worst cases, restore from backup.
- **CDN-cached assets:** cache purge plus deploy of old build.
- **Auto-deploys:** wait for next poll cycle to pick up the revert.

Asymmetric rollback means: if you need to roll back, the order may be the *reverse* of the deploy order, and the latency may be different. Plan the rollback before you ship, even if you never need it.

## Common multi-target shapes (templates)

### Two-repo dance, one auto-deploy

```
Targets:
1. backend-repo — auto-deploys from origin/main poll, ~2min
2. frontend-repo — manual push, no auto-deploy, served by static host

Sequence:
1. Pre-check both: type-check, lint, smoke-test.
2. Push backend-repo first (auto-deploy fires).
3. Wait for backend deploy to land (poll the version endpoint or deploy marker).
4. Push frontend-repo.
5. Verify frontend can talk to backend.
```

### Migration + code

```
Targets:
1. database — migration in supabase/migrations/<N>.sql, applies via dashboard or CI
2. backend — code that reads the new column

Sequence:
1. Migration must be backward-compatible (additive column with default).
2. Apply migration, verify schema.
3. Deploy backend code that reads the column.
4. (Optional, separate change) Stop writing the old column; later, remove it.
```

### Package + consumer

```
Targets:
1. cli-package — npm publish at new version
2. backend-repo — bumps cli-package dep

Sequence:
1. Pre-check: backend's tests pass with the new cli locally (npm link or local install).
2. npm publish cli-package@<new-version>.
3. Verify on registry: npm view cli-package@<new-version>.
4. Bump backend's package.json dep, push.
5. Verify backend deploy succeeds with the new version.
```

## Anti-patterns to refuse

- **All-at-once deploys.** "Push everything, hope for the best." Misorderings turn into 503 cascades.
- **Skipping verification between targets.** "I'll just verify at the end." If target 2 broke, target 3 may make it worse or block rollback.
- **Treating auto-deploy as instant.** Auto-deploy polls have latency. Pushing target B that depends on target A's deploy completing is a race.
- **Coupled changes across targets without a coordinating commit message.** Future-you trying to revert needs to find the matching commits; without explicit cross-references, this is forensic work.
- **Reverting one target without considering the others.** A revert in target 1 may force a coordinated revert in target 2; otherwise you're left with the failure mode the deploy was trying to avoid.
- **Pushing without staging-equivalent testing.** Multi-target changes have integration paths that single-target tests miss. If your project lacks staging, the test environment is production — be slower, not faster.

## Pairs with

- `pre-ship-adversary` — the adversary attack should explicitly include "what if only one target deploys?"
- `incident-recovery` — partial-deploy state often produces a freeze pattern; the recovery skill applies to the partial-deploy case.
- `first-time-action-gate` — first-time deploys to any target carry extra ceremony; this skill is the routine-case sequencing.
- Your project's deploy runbook (justfile, makefile, scripts) — the runbook encodes the targets and ordering for one project; this skill is the pattern when no runbook exists yet.
