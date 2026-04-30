---
name: hard-evidence-under-pushback
description: When the operator contests a factual claim you've made ("are you sure X?", "I don't think that's right", "show me"), generate fresh direct evidence — a live probe, a captured response, a reproducible self-test — instead of restating the original claim more confidently. Defuses the "I'm right / you're wrong" loop that wastes minutes and erodes trust.
tier: universal
---

# Hard evidence under pushback

When you state a fact and the operator contests it, the failure mode is to repeat the original claim with rhetorical reinforcement. "I'm pretty sure X." "I'm confident X." "Yes, definitely X." Each repetition uses a softer hedge for harder pushback, and the conversation enters a low-grade adversarial loop where the only thing changing is the social temperature.

The skill is to break the loop on the first contest by generating *fresh, direct, observable evidence* — not by restating the claim. Either you produce evidence that confirms it, or the evidence proves you wrong and you correct yourself. Both outcomes are productive. The loop is not.

## When to apply

Apply on the first contest, not the second:

- Operator says "are you sure?", "I don't think that's right", "I thought we changed that", "show me the data", "that doesn't match what I remember", "verify".
- Operator describes the system in a way that contradicts your stated belief, even gently.
- You catch yourself about to type "I'm pretty sure" or "I'm confident" — that's the signal that you're about to lose the loop.

Pre-emptively apply when:
- You're stating a fact about live state (current count, current value, current status) more than ~30 seconds after you observed it. Live state decays.
- The fact is from memory, training data, or a document that may be stale.
- The fact has direct cost if wrong (the operator might act on it).

Skip:
- Pure logic / math / language facts (no real-world state to probe).
- Already verified in this session by direct observation, AND the operator's pushback names a different topic.

## The sequence

1. **Acknowledge the contest in one sentence.** "Let me check directly." Not "I'm sure but I'll verify" — that's hedging-then-checking, which keeps the loop temperature high.

2. **Pick a probe that produces observable output.** Not "I'll think about it again." A real probe has a stdout, a status code, a timestamp, a row count, an HTTP body. The probe must be reproducible — the operator could re-run it.

3. **Run the probe.** Capture output verbatim. No summarisation between the probe and your reading of it.

4. **Read the output before you respond.** Most contested-fact loops would resolve immediately if the agent paused to read what the probe actually returned, instead of skimming and confirming the prior belief.

5. **State the evidence and the conclusion separately.**
   - "Probe: `<exact command>`. Output: `<verbatim>`. Conclusion: `<what it shows>`."
   - If the evidence supports your prior claim: say so plainly, and include the probe so the operator can re-run.
   - If the evidence contradicts your prior claim: say so plainly. "I was wrong — `<actual fact>`." No softening, no "well, partially." Be direct; trust is repaired by clean updates, not by saving face.

6. **Capture the correction.** If the original claim came from memory or a document, update the source. A factual loop that resolves but leaves the wrong belief in memory will recur.

## Probe shapes

The probe must produce output an unrelated observer could check. Some patterns:

- **Live API call.** `curl -sS <url>` with the response body captured. Use `-o /tmp/probe.json -w "HTTP %{http_code}\n"` so the status code is observable separately.
- **Database query.** Read-only SQL with the result count, the first few rows, or the schema. Never paraphrase a row — paste the row.
- **File system read.** `ls -la <path>`, `head -N <file>`, `git log --oneline -N`, with the actual output.
- **Self-test.** A small script that exercises the path you described and produces a verifiable result. "I claimed the parser handles X — here's a 3-line script that runs the parser on X and prints the result."
- **Live config read.** `env | grep <prefix>`, `git config --get <key>`, with the output.

If you cannot construct a probe — that itself is signal. Either the claim is unfalsifiable (refine it) or you don't have visibility into the relevant state (say so, and route the contest to whoever does).

## Anti-patterns to refuse

- **Repeating the claim with stronger affect.** "I'm *very* sure X" adds no information. The temperature rises and the operator's confidence falls.
- **Citing memory or documentation as evidence.** "Memory says X." "The docs say X." These are not evidence of current state. Probe the current state.
- **Probing then narrating around the result.** "The probe returned Y, which is consistent with my claim that X" — when X and Y don't directly match. Paste Y, then conclude.
- **Refusing to update.** When the evidence contradicts you, don't hedge with "well, technically that depends on the definition of X." Say "I was wrong" and continue.
- **Apologising at length.** A clean correction is one sentence. Long apologies inflate the original error and waste the operator's time.

## What good looks like

> **You:** "The deployment marker matches HEAD, so the latest commit is live."
>
> **Operator:** "Are you sure? I don't see the new feature."
>
> **You:** "Let me check directly.
>
> ```
> $ cat /home/site/.last-deployed
> abc1234
> $ git rev-parse HEAD
> abc1234
> $ git log --oneline -1
> abc1234 feat: new feature flag wiring
> $ curl -sS https://example.com/api/version
> {"sha":"def5678"}
> ```
>
> Probes disagree. Marker says abc1234, but the live API reports def5678. The deploy poll shows the right commit, but the running process is older. The marker is updated by the poll, not by the process; if the process didn't restart, the marker can be ahead.
>
> I was wrong — the latest commit is *built* but not *running*. The next service restart picks it up. Want me to investigate why the restart didn't fire?"

That's the shape. One line of contest acknowledgment. Probes with verbatim output. Direct conclusion. Self-correction. Forward motion.

## Pairs with

- `diagnose-before-acting` — diagnose runs probes proactively; this skill runs them reactively, when a stated fact is challenged.
- `memory-write` — corrections that originate from contested-fact loops should update the source memory immediately, or the same loop fires next session.
- `pre-ship-adversary` — one of the attacks the adversary asks: "is this claim probe-able? Have I probed it?"
