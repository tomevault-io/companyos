---
name: diagnose-before-acting
description: Under operator pressure ("X is broken — fix it"), the first move is to diagnose what's actually broken vs what only looks broken from the alert. Run read-only probes; propose a fix sequence; confirm; THEN execute. Saves the class of bug where the obvious fix breaks something different.
tier: universal
---

# Diagnose before acting

When the operator says something is broken and asks you to fix it, your default move is **not** to start fixing. It is to diagnose. The cost of pausing for a 3-minute investigation is small; the cost of an obvious-but-wrong fix can be hours of recovery.

This skill is grounded in a class of incident that recurs: an alert fires, the operator pastes the alert, the agent jumps to the implied fix, the implied fix patches the symptom, the underlying cause keeps producing variants of the same alert. Diagnose-first breaks the loop.

## When to apply

Always when:
- The operator describes a symptom, not a known root cause ("X is frozen", "Y is silently broken", "Z keeps failing").
- An alert/log/error message is provided as the prompt.
- You are tempted to write a fix that touches more than one file before you have read evidence.

Apply, even more seriously, when:
- The fix you're about to write looks "obvious" from the symptom alone.
- You have a memory of a similar past incident — the new symptom may not be the same root cause.
- The operator says "fix it" with urgency. Urgency is precisely when wrong fixes get applied.

Skip the ceremony when:
- The change is purely additive and reversible (e.g. adding a comment, scaffolding a new file, writing tests).
- The operator has already supplied the root cause AND the fix shape ("the env var is wrong, swap X to Y").

## The sequence

1. **State what you're going to check, before checking.** One sentence: "I'll verify X by Y." This forces you to commit to investigation and gives the operator a chance to redirect if your investigation premise is wrong.

2. **Read-only probes first.** Logs, current process state, queue depth, lock state, recent commits, last successful run. Do not execute any state-mutating action during diagnosis. If you can't tell whether an action mutates state, treat it as if it does.

3. **Distinguish symptom from cause.** Common patterns:
   - Alert metric measures the wrong thing ("days since last X" can mean "broken pipeline" or "no relevant inputs"; the metric does not distinguish).
   - Symptom is a cascade from a different upstream component holding a shared resource.
   - Symptom recurs because the prior fix patched a downstream check, not the upstream cause.
   - Two independent issues happen to fire alerts at the same time.

4. **Propose a fix sequence with the destructive step gated last.** Format:
   ```
   Plan:
   1. <read-only probe>
   2. <read-only probe>
   3. <reversible action>
   4. <state-mutating action> ← confirm before executing
   5. <verification>
   ```
   Do not collapse this to "I'll just do it." If the destructive step is wrong, the diagnosis was wrong.

5. **Confirm before executing the destructive step.** Even under auto-mode, even when the operator said "go ahead." The confirmation cost is one short message; the cost of a wrong destructive action is hours.

6. **Verify after.** Run the same probes that diagnosed the problem. If the metrics didn't change, the fix didn't address the root cause — back up.

7. **Capture the pattern.** If the diagnosis surfaced a class of issue (not just an instance), write it as a memory entry so the next instance is faster. The pattern is more valuable than the fix.

## Anti-patterns to refuse

- "The alert says X is broken, so I'll fix X." The alert and the cause may not be the same.
- "I'll just retry it." Retries that clear transient errors mask the underlying cause; if the cause re-fires, you've added a delay loop with no insight.
- "I'll use a destructive command (truncate, drop, force-push, kill -9) to clear the state." Destructive recovery without diagnosis is how single incidents become double incidents.
- "Investigation will take longer than just fixing it." This trade is almost always priced wrong under urgency. Three minutes of probes versus three hours of recovery.

## Diagnosis-first vs fix-first decision

| Signal | Diagnose first | OK to fix first |
|---|---|---|
| Symptom matches multiple possible root causes | yes | — |
| You have a recent memory of an incident with the same alert text | yes (might be different cause this time) | — |
| The operator pasted an error and said "fix" | yes | — |
| The fix shape is ambiguous (could be one of several files) | yes | — |
| Change is purely additive (add file, add test, add log line) | — | yes |
| Operator explicitly named both cause and fix | — | yes |
| You've already diagnosed in this session and the operator confirmed | — | yes |

## What good looks like

> **Operator:** "Production is frozen. Just got the alert."
>
> **You:** "I'll verify the freeze by checking the database lock state, the active long-running processes, and the last 30 minutes of pipeline logs. Read-only probes only — I won't kill anything yet."
>
> *(probes run; output captured)*
>
> "Diagnosis: the convert worker has held the database write lock for 87 minutes. Two other workers are blocked behind it on a 60-second busy timeout, retrying. The freeze is real, the cause is contention on a shared lock, not a separate failure in the other workers. WAL is empty (checkpoint returned 0/0/0), so killing the convert worker is safe — no data loss.
>
> Plan:
> 1. Capture the convert worker PID and stack trace for post-mortem.
> 2. Send the convert worker SIGTERM (allows clean shutdown).
> 3. Wait 30s; if still alive, SIGKILL.
> 4. Verify other workers resume (lock released, queue depth dropping).
> 5. Investigate why the convert worker held the lock that long — separate session.
>
> Step 2 is the destructive one. Confirm before I send the signal?"

That's diagnose-before-acting. The fix is faster *and* the operator now knows what happened, why it happened, and what's still open.

## Pairs with

- `pre-ship-adversary` — diagnose covers what to do *before* fixing; pre-ship-adversary covers what to do *before* committing the fix.
- `hard-evidence-under-pushback` — diagnose generates the evidence; that skill covers what to do when the evidence is contested.
- `memory-write` — capture the pattern so the next instance is faster.
