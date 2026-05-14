---
name: grill-me
description: ALWAYS run this skill FIRST when the operator describes a new feature, surface, schema change, distribution mechanic, redesign, refactor, or any non-trivial cross-cutting work — BEFORE Superpowers brainstorming, BEFORE writing-plans, BEFORE any spec, briefing, or implementation. Interview the operator relentlessly, one question at a time, walking down each branch of the design tree and resolving dependencies between decisions one-by-one. For every question, provide your recommended answer. If a question can be answered by exploring the codebase, explore the codebase instead of asking. The goal is a shared design concept BEFORE any artifact (plan, PRD, code) is created. Triggers on phrases like "let's build X", "I want to add Y", "design a Z", "we should change Q", "new feature", "redesign", "rebuild", "schema for", "should we restructure", "rework the", "plan for", "approach for", "how should we". Skip ONLY for explicit trivial work the operator scoped tightly ("typo fix", "rename one var", "small copy tweak", "single-line bump"). When in doubt, grill — the cost of grilling is ~5 minutes, the cost of pivoting after shipping is ~1 week.
metadata:
  author: Matt Pocock (mattpocock)
  source: https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me
  license: MIT
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

## TomeVault use

This skill sits **upstream** of Superpowers `brainstorming` and `writing-plans`. It is the first thing to run on anything non-trivial — new feature, schema change, distribution mechanic, surface redesign, anything cross-cutting.

The TomeVault failure pattern this addresses: build → ship → realize design was off → pivot six weeks later. Examples in memory: Studio toggle → merged canvas (9649dc1), Tome individual-files → bundles, distribute → platform-coverage rename, factory build-new → evolve-only, roadmap_sync paused, Genome paused, Council retired, classifier 24/7 paused. Each of those would have been caught by a 20-minute grill-me before any code or briefing got written.

After grill-me, output options:
- For a build task → run `to-prd` to produce a `briefings/` PRD.
- For a strategic question → write the resolution into `.context/decisions.md`.
- For a small clarification → just proceed; grill-me's value was the shared concept, not the artifact.

## License

MIT, Matt Pocock. Original: https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me.
