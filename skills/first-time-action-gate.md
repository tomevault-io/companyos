---
name: first-time-action-gate
description: The first time you do any externally-visible action — first email batch, first marketplace push, first package publish, first production database migration, first webhook to a partner — apply explicit ceremony: dry-run, scoped test, operator confirmation, post-action verification, rollback awareness. From the second instance onward, the ceremony can relax to a routine check. The cost of the ceremony is small; the cost of a botched first action is permanent.
tier: universal
---

# First-time action gate

The first instance of any externally-visible action carries asymmetric risk. A botched first email batch trains the recipient list to mark you as spam — recoverable, but only over months. A botched first marketplace push lands public artefacts that are hard to retract. A botched first migration touches data that may not be backed up at the right granularity. After the second or third successful instance, the action is routine; the failure modes are known and instrumented.

This skill is the ceremony for the *first* of each class. It's deliberately heavier than the routine path. Once a clean first run lands, the action moves to your project's standard runbook.

## When to apply

Apply the gate, in full, the first time you:

- Send any email batch or notification campaign to recipients outside the team.
- Push artefacts to any external marketplace, registry, or directory (npm, PyPI, GitHub marketplace repos, vendor stores).
- Publish a package version (especially the very first publish of a new package).
- Apply a schema migration on any production-class database with real users.
- Open a pull request or issue against any external repository (especially under a creator or partner's namespace).
- Issue a webhook or API call to a partner system with side effects.
- Trigger a data backfill or batch update affecting more than a handful of records.
- Deploy any new automation that takes destructive action without per-instance human review.

Apply a relaxed version when:
- The action class has run cleanly N times and the failure modes are understood.
- The same action ran cleanly in a staging or test environment with the same shape as production.

Skip the gate when:
- The action is purely internal (no external visibility, no partner touch, no recipient).
- The action is recoverable trivially (e.g., a draft email, a test push to a private fork).

## The gate

### Phase 1 — Dry-run

Every gated action must have a dry-run mode that does everything except the destructive step.

- **For sends:** render the message, run the audit pipeline, log the would-have-sent records, do not invoke the SMTP or API send. Inspect the dry-run output the same way you'd inspect the live output.
- **For pushes:** check out the target repo, generate the diff, validate the file contents, do not push. Diff is reviewed before the push step is unblocked.
- **For migrations:** run against a non-production copy of the schema (test database, snapshot restore). Verify the up- and down- migrations both work.
- **For webhooks:** fire at a request bin (or local listener) that captures the payload and verify the payload shape matches expectations.

If the action class has no dry-run mode, **build one before doing the action**. Building dry-run capability is a one-time cost that pays back on every future first-of-its-kind action.

### Phase 2 — Scoped first run

After dry-run is clean, do the smallest live run possible.

- **For sends:** 1 to 5 recipients, all under your direct control (your own addresses, team members who've consented to be in the test). Verify deliverability, spam-folder placement, link tracking, unsubscribe behaviour, reply path.
- **For pushes:** 1 target, ideally a repo you own, with a clear "this is a first-run test" marker (commit message, branch name, labelled tag).
- **For migrations:** apply to one row, one table, or a small subset, where the rollback is unambiguous if the result is wrong.
- **For webhooks:** fire at one partner endpoint with a low-stakes payload, verify acknowledgement, verify the partner's downstream system shows expected effects.

A scoped first run gives you:
- Real-world signal the dry-run can't simulate (deliverability, partner system behaviour, network conditions).
- A small enough blast radius that mistakes are recoverable.
- A reference point to validate the at-scale run against.

### Phase 3 — Operator confirmation before scaling

Even when the scoped run is clean, the gate's third phase is explicit operator confirmation before scaling. Format:

```
First-time-action gate — <action class>

Dry-run: <result>
Scoped run: <N> recipients/targets/rows, <result>
Validation: <how I confirmed the scoped run worked>
Plan to scale: <to N total, in batches of M, with intervals of T>
Rollback: <how to revert if the scaled run misbehaves>

Confirm to scale, or pause to investigate.
```

The operator's role is judgment — the dry-run and scoped run cover technical correctness; the operator covers strategic correctness ("is now the right time to send to 1,000 people?").

Do not collapse this phase to assumed approval. Even when the operator said "go ahead" in the prior message, the dry-run / scoped-run / scale gates are checkpoints, not formalities.

### Phase 4 — Scaled run with monitoring

When the operator confirms, run the scaled action with explicit monitoring:

- **Observability ready:** the action's success and failure surfaces (email logs, partner webhooks, deploy markers, error rates) are being watched in real time, not checked after the fact.
- **Circuit breakers wired:** for sends, a complaint or bounce rate threshold that aborts the batch. For pushes, a rate limit or error count that pauses. For migrations, an exception count that triggers rollback.
- **Pacing:** scaling N from 5 to 1,000 in one step is rarely the right move. Intermediate steps (50, 200, 500) catch latent failure modes that small samples don't.

### Phase 5 — Post-action verification

After the scaled run completes:

- **Sample the output.** Read 5-10 actual results (delivered emails, pushed artefacts, migrated rows). Don't just check the summary count.
- **Check downstream effects.** Did the action do what it was supposed to do for the recipient/partner/data? An email that delivered but read as spam is a successful action with a failed outcome.
- **Capture the baseline.** This run is the reference for the second-time and routine-time runs. Note the metrics: deliverability rate, error rate, latency, downstream conversion.

### Phase 6 — Promote to routine

After the first run is verifiably clean, the action class can move to your project's standard runbook. Promote means:
- Add a recipe / script that captures the production sequence.
- Document the routine pre-checks (often a strict subset of the first-time gate).
- Note when the first-time gate must re-apply: a major schema change in the action's payload, a partner change, a new recipient class.

Promotion is not "skip the ceremony." It's "the ceremony is now small enough that a human-or-agent can run it confidently."

## Action-class examples

### First email batch
- Dry-run: render every email, audit (URL count, vocabulary, banned formatting), log to a JSONL file, do not send.
- Scoped: 5 emails to operator-controlled addresses across providers (gmail, yahoo, outlook).
- Confirmation: deliverability, spam folder check, click tracking, reply path test.
- Scaled: in batches with circuit breaker on bounce/complaint rate.
- Routine: same audit pipeline, no operator confirmation per send.

### First marketplace push
- Dry-run: prepare the artefact in a staging area, validate format, render the README that will appear.
- Scoped: push to one repo you own, verify the artefact resolves correctly from a clean install.
- Confirmation: visual check of the public listing, install/install-from-URL test.
- Scaled: roll across remaining repos with rate limit and error abort.
- Routine: pre-push validation script; no operator confirmation per push.

### First package publish
- Dry-run: `npm pack` (or equivalent) to inspect the tarball; check `files`, `bin`, `exports` fields.
- Scoped: publish to a scoped or staging registry, install from there, run the binary.
- Confirmation: real-world install in a clean environment.
- Scaled: publish to the main registry.
- Routine: version bump + pre-publish smoke test; the publish itself doesn't need a per-instance gate.

### First production migration
- Dry-run: apply against a snapshot copy.
- Scoped: apply to one shard / one table / one tenant, verify behaviour.
- Confirmation: production smoke test on the scoped subset.
- Scaled: apply to the rest with monitoring.
- Routine: schema migrations remain operator-reviewed when they touch live tables; the gate stays partially active for this class.

## Anti-patterns to refuse

- **Skipping the dry-run because "it's a small change."** Small first-of-its-kind changes have the same blast radius shape as large ones; the cost is in the lack of a baseline, not the size of the diff.
- **Conflating dry-run with scoped run.** Dry-run validates *the code path*. Scoped run validates *the real world*. Both are needed.
- **Self-approving the scale step.** The whole point of the operator gate is that the agent and the operator have different signals. Self-approval throws away that signal.
- **Treating the gate as bureaucracy to be minimised.** The gate exists because the failure modes are real. The cost of one botched first-time action — a deliverability reputation hit, a public retraction, a data corruption — is much larger than the gate's overhead.
- **Promoting to routine after one run.** Two clean runs is the minimum before promotion; three is more honest. One could be luck.
- **Skipping post-action verification because the metrics looked good.** Metrics summarise; sampling reveals. Always sample.

## Calibration signals

Healthy gate discipline:
- The first run of a new action class produces an artefact (a runbook, a recipe, a memory entry) that documents the action for future runs.
- The number of action classes that have been promoted to routine increases steadily.
- First-time actions surface real failure modes in dry-run or scoped run, not in production.

Unhealthy gate discipline:
- "First-time action" actions that have run dozens of times silently — gate stopped applying years ago, no one noticed.
- Routine actions that should have a gate (because something material changed) but don't.
- Scoped runs that look like dry-runs (too small to validate; too cautious to be useful).

## Pairs with

- `pre-ship-adversary` — the adversary attack runs alongside the dry-run; both are looking for what could go wrong before the action fires.
- `decision-memo` — when the action class is significant, write a memo before the first run capturing the rationale and the rollback plan.
- `multi-target-publish` — first-time multi-target deploys carry both gates; apply both.
- `memory-write` — the post-run baseline goes into memory so the second run can compare.
