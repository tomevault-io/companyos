# TomeVault Adaptation Notes

> Read alongside `SKILL.md`. Documents how this skill applies to TomeVault specifically.

## Source

Matt Pocock — MIT. Original at https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture.

Copied verbatim. The 4 files (`SKILL.md`, `DEEPENING.md`, `INTERFACE-DESIGN.md`, `LANGUAGE.md`) are unchanged. This file is the only TomeVault addition in this directory.

## When to fire this skill

**Manual invoke only.** Not auto-triggering.

Specifically invoke when:

- We've shipped enough new code in `tomevault-web/src/` or `scripts/lib/` that a sweep for shallow-module rot is worth the cost (~quarterly).
- A specific surface has become hard to change without bugs (the operator notices, or a refactor proposal keeps stalling).
- After a major feature lands and we want to consolidate the architecture before the next big push.

**Do NOT invoke** in scope-creep scenarios. If we're in the middle of a small task, log architectural observations to `briefing-divergences.md` per `GUARDRAILS.md §2 "Scope Discipline"` and continue. Don't sidetrack into a refactor mid-task.

## Where to apply it

The TomeVault codebase has different surfaces with different risk:

| Surface | Audit value | Notes |
|---|---|---|
| `scripts/` (the 6-script pipeline) | **Low** | Already deep-module-shaped. Each script is one big, well-bounded thing with a simple CLI interface (`python3 discover.py --env test`). Don't refactor unless something specific is broken. |
| `scripts/lib/` | **High** | Where shared logic accretes. Prime candidate for deepening — modules like `classifier.py`, `email_sender.py`, `asset_layer.py` may have grown shallow over time. |
| `tomevault-web/src/` | **High** | React/Next.js code is the easiest place to grow shallow modules — lots of small components, lots of pass-through types, lots of one-line "utility" files. Audit `tomevault-web/src/lib/`, `tomevault-web/src/components/`, and the API route handlers. |
| `tomevault-web/src/app/` | **Medium** | App Router pages and layouts. Worth checking but the structure is somewhat dictated by Next.js conventions. |
| `paperclip-configs/`, `factory/` | **Low** | Configuration-as-code. Audit only if a config refactor is already planned. |

## Where Matt's skill references things we don't have

### `CONTEXT.md` (Matt's ubiquitous-language file)

Matt's skill expects a `CONTEXT.md` at the repo root with the project's domain vocabulary. **We don't have one.**

TomeVault equivalents to consult instead:

1. `CLAUDE.md` — project identity, owned terms, conventions.
2. `tomevault-design/components/voice-and-copy.md` — vocabulary discipline + owned terms (Tome, Vault, Standards, Studio, Relay, Genome, Skill, claim).
3. The memory index — strategic terms (Helm+Snyk metaphor, OWASP-2001, etc.).

When the skill says "use the terms from CONTEXT.md," read those three sources and use *their* terms consistently.

### `docs/adr/` (Architecture Decision Records)

Matt's skill respects existing ADRs. **We don't have an ADR directory.** TomeVault equivalents:

1. `.context/decisions.md` — strategic + architectural decisions with rationale.
2. `.context/patterns.md` — gotchas, conventions, lessons learned.
3. `briefing-divergences.md` — deviations from briefing specs.
4. Memory entries with `type: project` and `type: feedback` — durable decisions.

Consult all four before proposing a deepening that contradicts a recorded decision.

## Output expectations

When this skill produces deepening proposals, write them to `briefings/<YYYY-MM-DD>-architecture-<scope>.md` — same convention as `to-prd`. Don't just dump suggestions into chat; the audit value is durable if it's a briefing, ephemeral if it's not.

For each proposed deepening, the briefing should include:

- The current shallow-module pattern (with concrete file paths).
- The deletion-test result (what happens if we delete the module).
- The proposed deep-module interface.
- The migration approach (incremental, behind a seam, with a rollback path).
- The acceptance criteria (what makes the deepening "done").

## What NOT to do

- **Don't propose deepenings to the pipeline scripts.** They're already deep. Mucking with them creates risk for no gain.
- **Don't propose deepenings to `scripts/factory/` mid-evolve-only-pivot.** Factory state is in flux per memory (`project_factory_evolve_only`); architectural changes there should wait until the pivot stabilizes.
- **Don't propose deepenings to anything explicitly marked frozen** in CLAUDE.md or memory (Standards page, staging environment build plan, etc.).
- **Don't refactor for "AI navigability" alone.** That's a fine secondary benefit but not a primary driver. If the operator can't easily change the module today, that's the driver. If they can, leave it.

## License

MIT, Matt Pocock. See `LICENSE` in this directory.
