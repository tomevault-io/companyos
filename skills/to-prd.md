---
name: to-prd
description: Turn the current conversation context into a PRD saved as briefings/YYYY-MM-DD-slug.md in TomeVault. AUTO-INVOKE this skill immediately after grill-me resolves on a buildable task — do not wait to be asked. Also invoke when the operator says "let's write this up", "spec this out", "make a briefing", "PRD this", "document the plan", or signals the design conversation has concluded and implementation is next. Do NOT interview the operator (grill-me did that); just synthesize what you already know about the codebase + conversation + decisions and produce the briefing. Skip ONLY for: trivial work (typo, one-liner), pure strategic decisions with no build component (those go to .context/decisions.md instead), or when an existing briefing already covers this scope (update that one).
metadata:
  author: Matt Pocock (mattpocock) — adapted for TomeVault
  source: https://github.com/mattpocock/skills/tree/main/skills/engineering/to-prd
  license: MIT
---

# to-prd

Takes the current conversation context and codebase understanding and produces a PRD as a markdown file in TomeVault's `briefings/` directory. Do NOT interview the user — that's `grill-me`'s job. By the time this skill fires, the shared understanding should already exist.

## TomeVault adaptation

Matt's original skill publishes the PRD to an issue tracker with a `ready-for-agent` triage label. TomeVault doesn't use issues as the work surface (roadmap_sync auto-ticketing is paused per memory; RDMP items + briefings + decisions.md are the shared backlog). Instead:

- **Output location:** `briefings/<YYYY-MM-DD>-<kebab-slug>.md`
- **Filename example:** `briefings/2026-05-11-studio-share-modal.md`
- **No labels.** Status tracking happens via filename / commit / divergence log.
- **Domain glossary:** TomeVault's vocabulary (Tome, Vault, Standards, Studio, Relay, Genome, Skill, claim, etc.) per `tomevault-design/components/voice-and-copy.md` and the memory's "Owned terms" entries. Respect these terms; don't drift.
- **ADRs:** TomeVault doesn't keep ADRs in `docs/adr/`. Equivalent records:
  - Strategic decisions → `.context/decisions.md`
  - Architectural / pipeline patterns → `.context/patterns.md`
  - Briefing divergences → `briefing-divergences.md`

Consult these before writing the PRD to ensure consistency with prior decisions.

## Process

1. **Explore the repo** to understand the current state, if you haven't already. Use TomeVault's domain vocabulary throughout the PRD. Respect any existing patterns in `.context/patterns.md` and prior decisions in `.context/decisions.md`.

2. **Sketch the major modules** you'll need to build or modify. Actively look for opportunities to extract **deep modules** that can be tested in isolation.

   A deep module (per "A Philosophy of Software Design") is one that encapsulates a lot of functionality behind a simple, testable, rarely-changing interface. Shallow modules have interfaces nearly as complex as their implementations.

   Check with the user that these modules match expectations. Check which modules they want tests written for.

3. **Write the PRD** using the template below. Save it to `briefings/<YYYY-MM-DD>-<kebab-slug>.md`.

4. **Update `.context/state.md`** to reference the new briefing if it's a notable piece of work.

5. **Commit the briefing** with a conventional commit prefix (`docs:` for the briefing itself, or `feat:` if it ships alongside code).

## PRD template

```markdown
# <Feature name>

> Briefing for <one-line summary>. Authored <YYYY-MM-DD>.

## Problem statement

The problem the user is facing, from their perspective.

## Solution

The solution to the problem, from the user's perspective. Specific. No marketing filler.

## User stories

A long, numbered list. Each in the format:

1. As an <actor>, I want a <feature>, so that <benefit>.

Be extensive. Cover the obvious cases AND the edge cases AND the cases that come up during a grill-me but get forgotten by the time someone builds it. If the list is short, you missed cases — go back and find more.

## Implementation decisions

A list of decisions that were made. May include:

- The modules that will be built or modified
- The interfaces of those modules (what callers see, what's hidden inside)
- Technical clarifications from the operator
- Architectural decisions
- Schema changes (with reference to the migration that ships them)
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets — those will rot. **Exception:** if a prototype produced a precise artifact (state machine, reducer, schema, type shape) that encodes a decision more cleanly than prose, inline the relevant snippet and note its prototype origin. Trim aggressively; not a demo, just the decision.

## Testing decisions

- What makes a good test for this surface (test external behavior, not implementation details).
- Which modules will be tested. Which won't, and why.
- Prior art for the tests (similar tests already in the codebase).
- Integration vs unit: TomeVault prefers integration tests against real Meili / Resend / GitHub sandboxes over heavy mocking. See CLAUDE.md §"Tests must exist but writing order doesn't matter."

## Out of scope

What's explicitly NOT part of this PRD. Future work. Adjacent improvements that someone might think belong here. The "no, we're not doing X yet" list.

## Further notes

Anything else worth recording — open questions, design alternatives considered and rejected, references to prior briefings or memory entries.
```

## When NOT to use to-prd

- The work is trivial (a one-line fix, a copy edit, a typo). Just do it.
- A briefing already exists for this scope. Update or amend that one; don't fork.
- The shared concept isn't actually shared yet. Go back to `grill-me`.

## License

MIT, Matt Pocock. Original: https://github.com/mattpocock/skills/tree/main/skills/engineering/to-prd. Adapted for TomeVault's briefing convention and absence of an issue tracker.
