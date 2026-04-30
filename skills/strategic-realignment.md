---
name: strategic-realignment
description: When the operator says "what's next?" / "review the plans" / "where are we?" — the question is rarely "list our open tickets." It's "help me see the shape of the next arc." The skill is to cross-ground reality (recent commits, live state, observed metrics) against the canonical strategy docs, surface the genuine drift, present 2-3 named options with tradeoffs, and let the operator decide. Don't decide for them; don't list everything; don't restate the strategy.
tier: universal
---

# Strategic realignment

The operator asking "what's next?" is doing strategic thinking, not asking for a queue dump. They have a felt sense that the current direction needs revisiting. The skill is to translate that felt sense into a concrete reframe they can react to — not to enumerate work, not to decide for them, not to deliver a strategy lecture.

This is the highest-leverage conversational pattern in long-running projects. Done well, a 20-minute realignment redirects weeks of build. Done badly, it pads the operator's inbox with ten options they didn't want.

## When to apply

The operator initiates with phrasings like:
- "What's next?"
- "Let's review the plans."
- "Where are we?"
- "I feel like we should step back."
- "Is this still the right priority?"
- "Have we drifted from X?"
- "Help me see what we should be doing."

Or implicitly when:
- The operator's recent decisions show they're scope-questioning, not just executing.
- A meaningful event happened (incident resolved, gate signal moved, competitor moved, deadline shifted) and no one's reframed yet.
- The work has been heads-down for >2 weeks without a step-back.

Skip the ceremony when:
- The question is narrow ("what should I do *today*?"). Answer narrow.
- The operator already named the reframe and is asking you to cost it.
- A realignment happened in the last 7 days and nothing material has changed.

## The sequence

1. **Read the canonical strategy docs first.** Whatever your project's authoritative strategy file is — roadmap, vision doc, OKRs, north-star memo — load it before generating any options. Strategy lives there or nowhere; do not ad-lib direction from session context.

2. **Cross-ground against live state.** What does the strategy file say is current? What does reality say? Likely candidates for drift:
   - Stage/phase says X is current; build state says X shipped two weeks ago.
   - Strategy says we measure metric Y; metric Y is broken or missing.
   - A gate signal silently flipped (positive or negative) and no one noticed.
   - A risk on the watch-list became real and isn't reflected in the priorities.

3. **Read the recent operator-shaped artefacts.** Last session debriefs, last 30 commits, last decisions, recent feedback. The operator's thinking has been laid down in those — surface it.

4. **Identify the binding constraint.** What's actually limiting forward motion right now? Common shapes:
   - Volume — we need more of X (input, output, signal).
   - Quality — we have enough X but the quality is wrong.
   - Distribution — we have it but no one sees it.
   - Demand — supply is healthy, demand is cold.
   - Trust — the system works but readers don't believe it.
   - Attention — the work is right but the founder's time is mis-allocated.
   The binding constraint is rarely the same as the loudest alert.

5. **Present 2-3 named lanes, not a checklist.** Each lane needs:
   - A name that fits the strategic frame, not the implementation detail.
   - One sentence on why it matters now.
   - The honest tradeoff against the alternatives.
   - One concrete first move.
   Naming is load-bearing. "Lane 2 — lead the category" is reactable; "do these 17 things" is not.

6. **Recommend, then ask.** Pick one lane and say why. Be willing to be wrong — the operator's role is to redirect. Frame: "I'd start with Lane X because <constraint>; Lane Y is the alternative if <signal>."

7. **Don't decide.** The operator is doing strategic thinking. Your role is to surface the shape, present the tradeoffs, and recommend. The decision is theirs. Do not author plans, do not commit to a direction, do not start implementing — until the operator confirms which lane.

## What to NOT do

- **Restate the strategy.** They wrote it; they don't need it back. Reference, don't recite.
- **Dump the open ticket queue.** "What's next?" is not "what's pending?" Open tickets are the *output* of strategy, not the strategy.
- **Generate ten options.** The cost of ten options is paralysis. Three is the maximum useful number; two is often better.
- **Hide your recommendation behind neutrality.** "Either could work." If you have a view, state it. The operator wants pushback, not a menu.
- **Confuse activity with progress.** Long lists of recent commits are not strategic information. Strategic information is "we did X, the signal moved Y, now Z is the constraint."
- **Assume the strategy file is current.** Cross-grounding catches drift. The auto-generated state file may lag the actual state.

## Output shape

Roughly:

```
## Where we actually are
<one paragraph: the reality vs the canonical doc, naming any drift>

## The binding constraint
<one paragraph: what's actually limiting forward motion right now>

## Three lanes
### Lane A — <name>
<why it matters now; first move; tradeoff>

### Lane B — <name>
<same shape>

### Lane C — <name>
<same shape>

## My recommendation
<one paragraph: which lane and why; what would change my mind>

## Question
<one specific question to the operator, not "what do you want?">
```

Keep each section tight. The operator should be able to read the whole thing in 90 seconds.

## Calibration signals

After the realignment, watch for:
- The operator picks one of the named lanes — good; the framing was useful.
- The operator picks a fourth lane you didn't surface — moderate; you missed an option but the conversation moved.
- The operator pushes back on the framing itself ("you've got the constraint wrong") — useful; update and re-present.
- The operator asks for more detail before deciding — fine; provide it scoped to the chosen lane.
- The operator decides with no engagement — bad; the framing was probably wrong or trivial. Audit next time.

## Anti-patterns to refuse

- **Realigning every week.** Realignment fatigue is real. If nothing material has changed, the right response to "what's next?" is "Lane X (from last realignment) is still the right answer; here's the next concrete move."
- **Realigning to avoid the hard work.** Sometimes "what's next?" is dodging the current task. Notice if the operator's recent moves suggest they're stuck mid-execution; the right move may be helping them finish, not redirecting.
- **Authoring the next arc immediately.** A realignment surfaces the shape; the actual arc plan is a separate document the operator commissions after the realignment confirms the direction.

## Pairs with

- `decision-memo` — when the operator confirms a lane, the next move is often a decision memo on the specific change.
- `memory-write` — capture the binding-constraint reframe so the next realignment starts from the new baseline, not the old one.
- Any project's roadmap-amendment flow — if the realignment surfaces drift in the canonical strategy doc, draft an amendment.
