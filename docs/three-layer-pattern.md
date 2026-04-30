# The three-layer pattern

CompanyOS organises an AI-instructed company's operating context into three layers with strict separation:

```
┌──────────────────────────────────────────────────────────────┐
│  Constitutions       ←  operator-owned strategy               │
│  (read-only to agents; amendment flow for changes)            │
└──────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│  Justfile recipes    ←  human-or-agent procedures              │
│  (deterministic commands; one canonical place to find them)    │
└──────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│  Skills (markdown)   ←  agent-invoked patterns of thought     │
│  (situational; loaded when named conditions arise)             │
└──────────────────────────────────────────────────────────────┘
```

Each layer answers a different question:

- **Skills:** *"How should I think when X happens?"*
- **Justfile recipes:** *"How do I run Y?"*
- **Constitutions:** *"What is true about this company that doesn't change session-to-session?"*

The separation matters because the layers have different lifecycles, different governance, and different consumers.

## Skills — patterns of thought

A skill is a markdown file with frontmatter and a body. The agent loads the file when the situation it describes arises. The skill does not run; it informs how the agent runs.

Examples:
- "Before any non-trivial commit, attack the change yourself with these four attacks."
- "When the operator contests a fact, generate fresh evidence rather than restate."
- "When 'what's next?' arrives, ground reality against the strategy doc, present three named lanes, recommend one."

What makes a good skill:
- **Universal in shape.** Patterns that work for one company also work for the next.
- **Concrete in trigger.** "Use this when the operator says X" is more useful than "use this for important changes."
- **Honest about anti-patterns.** Most failure modes are systematic; naming them in the skill prevents the next instance.
- **Shorter than you'd expect.** The skill encodes judgment, not a procedure. If it's longer than ~250 lines, the pattern is probably actually two patterns.

What skills are NOT:
- Runbooks. ("Run command X then Y" lives in the justfile.)
- Strategy. ("This company prioritises Y over Z" lives in constitutions.)
- Documentation of how the system works. (That's in code comments and README files.)

## Justfile recipes — repeatable procedures

A justfile is a single file at repo root with a list of named recipes. Each recipe is a small command sequence. `just --list` shows the surface; `just <recipe>` runs it.

Why just specifically:
- **Discoverable.** One canonical place for "how do I run X?"
- **Self-documenting.** Each recipe has a description.
- **Cross-language.** Wraps Python, Node, Bash, SQL uniformly.
- **No daemons, no config rot.** A single binary, runs anywhere.

What makes a good recipe:
- **Idempotent if possible.** Re-running shouldn't hurt.
- **Side-effecting recipes flagged.** Anything that hits an external system, sends, deletes, or publishes carries explicit confirmation in the body.
- **Grouped.** `[group('health')]`, `[group('runbook')]`, `[group('smoke')]` so the list is readable.
- **One concern per recipe.** Composable. `just deploy` should call `just typecheck` and `just test`, not duplicate them.

What recipes are NOT:
- Patterns of thought. (Those are skills.)
- Strategic decisions. (Those are constitutions.)
- A replacement for CI. (Recipes are operator-runnable; CI runs them automatically. Different invocations of the same recipes.)

## Constitutions — operator-owned strategy

Constitutions are markdown files in a known directory (typically `constitutions/`) that describe what is *true* about the company that doesn't change session-to-session. Roadmap, design principles, non-goals, vocabulary discipline, mission.

Strict rules:
- **Operator writes.** Agents read.
- **Agents propose amendments via a separate file**, never edit the canonical document directly.
- **Amendment files have a structured format**: section affected, proposed change, why, confidence.
- **When the operator ratifies, they edit the canonical file** and move the amendment to `archive/` with a `-RATIFIED` suffix.

Why the separation:
- Strategy that drifts silently is the slowest, most expensive failure in long-running projects. Agents commenting on strategy is fine; agents *editing* strategy without operator review is not.
- The amendment trail itself is governance — anyone can see what was proposed, when, by whom, and why.
- Operators get a single place to look for "what is the company optimising for?" — not a session transcript reconstruction.

What constitutions ARE:
- Roadmap (with stage gates and signal-driven progression).
- Design principles (visual, brand, voice).
- Non-goals (the things you'll explicitly NOT do).
- Vocabulary (owned terms, banned substitutes, required triplets).

What constitutions are NOT:
- Open tickets or TODO lists. (Those are in your issue tracker.)
- Implementation details. (Those are in code.)
- Historical decisions. (Those are in the changelog or decision log.)

## Why the layers don't collapse

It's tempting to put everything in one place. Don't:
- **Skills in the justfile** become unrunnable command lists. The strength of skills is that they encode *judgment* the agent applies dynamically.
- **Recipes in skills** become unrunnable English descriptions of procedures. The strength of recipes is that they execute deterministically.
- **Constitutions in skills** become wishful documents that drift because no governance binds them. The strength of constitutions is the operator-write / agent-read split.
- **Skills in constitutions** become rigid rules that can't be evolved without operator-level ratification. The strength of skills is they evolve continuously through use.

The three-layer separation is not bureaucratic; it preserves *governance latency*. Skills evolve every session. Recipes evolve weekly. Constitutions evolve monthly or less. Letting fast-changing things contaminate slow-changing things is how strategy drifts.

## Memory as a separate concern

Memory is technically a fourth thing — agent-persistent observations across sessions. CompanyOS treats it as separate from the three layers because:

- Memory is *agent-written* (skills are usually operator-curated).
- Memory is *point-in-time* (constitutions are durable).
- Memory is *non-public* (skills and constitutions ship publicly).

The four-type memory taxonomy (user / feedback / project / reference) lives in `skills/memory-write.md`. The taxonomy is portable; the contents stay private.

## Bootstrap order

To adopt CompanyOS in an existing project:

1. **Add the skills.** Drop `companyos/skills/` into your agent-skills location. Adopt the AGENTS.md (or rename to your tool's preferred file). The skills start firing immediately.
2. **Add a justfile.** Copy `companyos/justfile.template` to your repo root. Replace placeholder recipes one at a time, in order of how often you run the command.
3. **Promote your strategy doc to a constitution.** Move it to `constitutions/<name>.md` and adopt the amendment-flow pattern. Even if you don't yet enforce read-only via tooling, the discipline matters.
4. **Wire the gates.** Pre-commit hooks for adversary attacks; CI checks for vocabulary violations; a hook blocking direct edits to constitutional files.

Order matters. Skills first because they're additive (don't break anything). Justfile second because it consolidates discoverability. Constitutions last because they require operator buy-in to enforce the read-only convention.

## Calibration

Healthy adoption:
- The agent loads skills without prompting in the named situations.
- `just --list` shows recipes that team members actually run, not aspirational placeholders.
- Constitutional documents accumulate amendments at a steady, slow rate (~1-2 per month is healthy; >5/month suggests the documents are too prescriptive; 0 over months suggests they're stale).
- Memory entries are referenced across sessions, not just written and forgotten.

Unhealthy adoption:
- Skills firing redundantly with overlapping triggers — usually means the patterns weren't tightly enough scoped at authoring.
- A justfile with recipes nobody runs — clean up.
- Constitutional documents that haven't been amended in a year — either the project is mature (good) or the documents have decayed into ignored boilerplate (bad). Re-read.
