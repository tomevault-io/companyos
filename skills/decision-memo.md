---
name: decision-memo
description: Produce a senior-leadership decision memo before proposing a non-trivial change. Seven sections — observation, hypothesis, proposal, evidence, blast radius, adversarial test, recommendation. Use when the change is non-obvious, affects shared state, or you want the operator to make an informed call instead of trusting your judgment.
tier: universal
---

# Decision memo

The operator may be a non-technical CEO, a senior engineer who needs the trade-off in plain English, or anyone who has to ratify your proposal without the time to reverse-engineer your reasoning. When you are about to propose anything non-trivial — a pivot, a significant refactor, a one-way architectural call, a commit of non-trivial scope — produce a memo first, then discuss. The memo format forces you to do the thinking before the asking.

## When to use

Auto-trigger for any of:

- Proposing a cross-cutting refactor (>3 files, or touching shared state).
- A migration, schema change, or anything that affects persistent data.
- An approach to a problem where two or more valid approaches exist and you're choosing between them.
- Any externally-visible action (sending, publishing, pushing) the project hasn't done yet.
- Anything the operator might reasonably push back on.

Skip for:

- Small, reversible edits (typo fixes, single-file bugfixes, isolated config tweaks).
- Direct instructions from the operator ("just do X") — the decision is already made.

## The memo

Produce this structure. Every section mandatory. Plain English throughout. No code identifiers above the "Appendix" heading.

```
## Headline
<one sentence naming the decision being asked for>

## 1. What I observed
<metric, signal, user behaviour, bug, constraint. Be specific about
volume, duration, source. 2-4 sentences.>

## 2. Hypothesis
<why this matters, what it suggests about the underlying system.
1-2 sentences. State confidence (almost_certain | likely | roughly_even
| unlikely | remote).>

## 3. Proposed change
<the specific change. If it's a diff, summarise in plain English — the
diff itself goes in the appendix. State reversibility: easy / moderate /
hard.>

## 4. Evidence
<data supporting the proposal. Comparable past changes and outcomes.
2-4 bullets. Be honest about what evidence is missing.>

## 5. Blast radius + rollback
<who/what this affects, who/what it doesn't. How to roll back if it
regresses. The specific signal to watch post-ship. 3-5 sentences.>

## 6. Adversarial test
<what you tried to break in the proposal. Specifically: scope creep
(does it affect more than claimed?), failure modes (null, empty,
duplicate, race), rollback rehearsal (is the stated rollback actually
reversible?), and shipping gate simulation (would your project's
allowlist or auto-ship guard, if you have one, accept this?). If any
attack lands, state the remediation or downgrade the recommendation.>

## 7. Recommendation
Ship / Don't ship / Pilot first.
Confidence: <almost_certain | likely | roughly_even | unlikely | remote>
<one sentence justification>

**Decision you're being asked to make:** <one sentence>

---

## Appendix (technical)

<diff, spec, file paths, SQL, event IDs, dependencies. Everything
the executor needs to act on the approved memo.>
```

## The four attacks (section 6)

Treat every attack as mandatory. Skipping an attack is a format violation — rewrite it, don't omit it.

1. **Scope creep.** Does the proposal affect anything beyond what section 3 claims? For config / filter / template changes, what reads the thing you're editing? Grep for consumers. Any behaviour change outside the memo's intent is a red flag.

2. **Failure modes.** Simulate unexpected input: null, empty, duplicate, race, upstream dependency down. State what the change does in each case. Undocumented behaviour is a downgrade signal.

3. **Rollback rehearsal.** Can the stated rollback actually revert? For DB changes, is there a non-destructive counter-query? For config changes, does the prior value still work? For code changes, does `git revert <sha>` produce a working state?

4. **Shipping gate simulation.** If your project has an auto-ship guard or release-gate (whitelisted scalar bumps, lint gates, schema-change reviews, deploy-policy rules) — would this proposal pass or block? A proposal that would pass is a low-stakes change; a proposal that would block is by definition operator-review-required. Be honest with yourself about which bucket this falls into. If your project has no such gate yet, ask whether one would have caught past failures — that is itself useful signal.

## After writing the memo

Present it to the operator. Wait for approve / reject / clarify. Do not execute until the operator has responded — writing the memo is the decision aid, not the decision.

The memo is shorter than you'd expect. If it's longer than ~30 lines above the appendix, you're specifying rather than deciding. Compress.

## Pairs with

- `pre-ship-adversary` — when committing a change, the four attacks of section 6 fire again as a pre-commit pass.
- `first-time-action-gate` — the memo precedes the gate when the action is first-of-its-class.
- `strategic-realignment` — a realignment surfaces *which* decisions need memos; the memo is the next step.
