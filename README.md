# CompanyOS — operating system in markdown

A complete, opinionated, fork-friendly operating system for an AI-instructed company. Three layers, in order of reach:

1. **Skills** — markdown patterns an AI agent loads to inform judgment. Decision memos, pre-ship adversary attacks, diagnose-before-acting under pressure, vocabulary discipline, memory writing. Thirteen skills, all universal.
2. **A justfile template** — a discoverable command surface for the *human-or-agent* runbook layer. Health checks, smoke tests, deploy dances, session-end rituals.
3. **Constitutional documents** — operator-owned strategy that agents read but never rewrite. An amendment-flow pattern keeps strategy and execution separate without bureaucracy.

The same 13 skills run inside the company that built this Tome. That's the dogfooding test: if a skill isn't useful enough for our own daily work, it's not useful enough to ship to anyone else.

## Why this exists

Most "company operating manual" content is one of two things:
- A blog post describing what one team does, not portable.
- A framework abstraction with no implementation, not actionable.

This Tome is the third option: a forkable artefact. The skills are written so they work for a company that has nothing to do with the one that authored them. The justfile is a *template* — your project's recipes live in your repo, not here. The constitutional pattern is a *shape*, not a content claim.

## What you get

```
companyos/
├── README.md                # this file
├── AGENTS.md                # root operating context (load at agent start)
├── CLAUDE.md → AGENTS.md    # alias for Claude Code
├── tome.json                # manifest (Tome spec v1)
├── justfile.template        # generic operator runbook starter
├── skills/                  # 13 markdown skills
│   ├── decision-memo.md
│   ├── pre-ship-adversary.md
│   ├── adversarial-test.md
│   ├── diagnose-before-acting.md
│   ├── vocabulary-check.md
│   ├── hard-evidence-under-pushback.md
│   ├── strategic-realignment.md
│   ├── memory-write.md
│   ├── incident-recovery.md
│   ├── multi-target-publish.md
│   ├── first-time-action-gate.md
│   ├── changelog.md
│   └── session-debrief.md
└── docs/
    ├── three-layer-pattern.md      # how skills + justfile + constitutions compose
    └── memory-taxonomy.md          # the 4-type memory pattern
```

## How to fork

1. Copy the `skills/` directory into your project's agent-skills location (`.claude/skills/`, `.cursor/rules/`, `.github/copilot-instructions/skills/`, depending on your tools — every major AI dev tool has its own location).
2. Adopt the `AGENTS.md` (or rename it to your tool's preferred filename — CLAUDE.md, .cursorrules, etc.). It's intentionally minimal; layer your project specifics on top, don't replace it.
3. Copy `justfile.template` to your repo's `justfile` and replace the placeholder recipes with your project's actual commands. The shape (groups, conventions, idempotency norms) is the contribution — the recipes are yours.
4. Read `docs/three-layer-pattern.md` once. The architectural decisions (why skills are markdown, why constitutions are operator-owned, why memory is separate from skills) inform everything.

That's it. The skills will start firing on the first session.

## What we kept private and why

This Tome is the *patterns*. The implementation behind it — the company's actual outreach genome, our specific search index schema, our growth-engine A/B logs, our memory contents — stays private because it's load-bearing competitive work. The pattern is the gift; our specific implementation is ours.

The honest line that goes here:

> "Buffer's open salaries didn't include their lead-scoring algorithm; Linear's changelog doesn't expose their pricing model decisions. We share the operating system, not the secret sauce."

If you fork this and want to share *your* operating system back, we'd love to see it. Open a PR or fork it under your own org. The shape generalises faster when more shapes exist.

## What's not here yet

- Project-specific runbooks (your company's freeze-recovery script, your deploy dance). The pattern is in `incident-recovery.md` and `multi-target-publish.md`; the recipes are in your `justfile`.
- A web UI for browsing skills. The Tome is intentionally markdown-first; surface it however you like.
- Quality / security scoring rules. Different companies have different threat models; pick your own.

## Versioning

Manifest at `tome.json` follows the Tome spec v1. Skill IDs are stable; descriptions and bodies evolve. Material breaking changes (rename, remove, shift in semantics) trigger a major version bump.

---

**Maintained by:** TomeVault.
**License:** MIT. Fork freely.
**Filing issues:** GitHub issues on this repo.
**Follow updates:** the Tome page on TomeVault for change-log + version history.
