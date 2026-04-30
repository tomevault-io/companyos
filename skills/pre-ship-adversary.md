---
name: pre-ship-adversary
description: Before committing a non-trivial change, spend 2-3 minutes attacking the change yourself — scope creep, failure modes, rollback, shipping-gate simulation. The cheap version of a formal adversary stress-test, internalised for direct coding. Catches the class of scope-creep bug where a config change is read by more consumers than the author intended.
tier: universal
---

# Pre-ship adversary

Before running `git commit` on a non-trivial change, attack the change like an adversary. Spend 2-3 minutes. Actually do each attack — don't just mentally wave at them.

This skill exists because of a recurring failure mode: an author edits a config / template / filter intending to change behaviour in one consumer. A *second* consumer also reads the file, with different semantics. The change ships; the second consumer's behaviour silently changes; production breaks in a way the change description does not predict. Three minutes of grep-for-readers would have caught it.

## When to use

Auto-trigger for:

- Any change touching a config file, filter list, blocklist, allowlist, or template that may be read by multiple consumers.
- Any change to side-effect-bearing code (anything that sends, pushes, or commits external state).
- Any change to pipeline control flow or worker orchestration.
- Any migration or schema change.
- Any `git commit` of more than ~30 added/modified lines in a file you don't own (first change you've made to it this session).
- Any change the operator said "be careful with".
- Any autofix or revert that you're about to commit.

Skip for:

- Pure documentation edits.
- Typo fixes.
- Adding test cases.
- Single-line scalar bumps (threshold tuning, version bumps) within already-guarded ranges.

## The four attacks

Run these in order. Each attack has a specific question and a specific check. If any attack lands, stop and either narrow the change or write a `decision-memo` instead of committing.

### Attack 1 — Scope creep

> Does the change affect anything beyond what I intended?

- Grep for consumers of the file(s) you edited. If it's a config, find what reads it. If it's a template, find where it renders. If it's a filter list, find everywhere the filter is applied.
- For each consumer, trace through: does my change do what I think in that consumer's context, or does it change behaviour there in ways I didn't plan for?

**Canonical failure mode.** A blocklist file edited to filter outbound recipients turns out to also be read by a sync job that purges matching documents from an index. The author intended one effect; production gets two, and the second one took the index offline. Greping for *every* reader of the file would have caught it. The grep should be the first action of this attack, not the last.

### Attack 2 — Failure modes

> What happens under unexpected input?

Run through the change's code path with:
- null / missing value
- empty list / empty string / zero
- duplicate firing (two runs back to back)
- race condition (two concurrent executions)
- upstream dependency down (database, third-party API, search index, external service)

You don't need to write code for each. Trace mentally. If the change silently swallows any of these, or produces an inconsistent state, fix it or document the tradeoff in the commit message.

### Attack 3 — Rollback rehearsal

> If this goes wrong, can I actually revert?

- For DB changes: is there a non-destructive counter-query? Could you recover data you just mutated?
- For config changes: does the prior value still work (or has the consumer changed in a way that breaks rollback)?
- For code changes: would `git revert <sha>` leave a working state, or does it produce orphan references / broken imports?
- For external state (email sent, PR pushed, webhook fired): rollback is impossible. You should have used `decision-memo` and `first-time-action-gate` instead.

### Attack 4 — Shipping-gate simulation

> Would this pass the project's auto-ship guard or release gate?

Most mature projects have *some* form of automated check that gates risky changes — a guard that blocks unauthorised paths, a lint pass that flags banned patterns, a CI job that requires human review on schema diffs. Run yours mentally (or actually):

- If your project has an explicit allowlist policy, run the diff through it.
- If it has CI rules that block certain paths or patterns, check whether your diff trips them.
- If you don't have any such gate, ask: would one have caught past failures in this code path? That's signal for whether one is worth building.

A proposal that would pass the gate is a low-stakes change; commit with confidence. A proposal that would block — or that touches a path you'd want a gate to block — is by definition operator-visible scope. Double-check the three attacks above and make the commit message explicit about why this is intentional. If in doubt, stop and write a `decision-memo` instead.

## After the attacks

If all four pass, commit. In the commit message, briefly note any attack that raised a flag and how you addressed it — a one-liner like "checked blocklist consumers: only the outreach module reads this; sync does not" is gold for the next person reading `git log`.

If any attack found a real issue, one of:
- Narrow the change to stay inside safe scope.
- Add tests that prevent regression of the attack you found.
- Escalate to a `decision-memo` and get operator sign-off.

Two-to-three minutes of adversarial thinking is cheap. A multi-hour outage is not.

## Pairs with

- `decision-memo` — section 6 of the memo runs the same four attacks before proposing; this skill runs them again before committing.
- `diagnose-before-acting` — diagnose covers what to do before fixing; this skill covers what to do before shipping the fix.
- `multi-target-publish` — when the change spans repos/packages/registries, run this attack on each target's diff separately.
