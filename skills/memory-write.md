---
name: memory-write
description: Persistent memory across sessions is one of the highest-leverage things a long-running agent can do — but only if you write the right things in the right shape. This skill defines the four-type taxonomy (user / feedback / project / reference), what NOT to save, and the structure each entry takes. Without discipline, memory becomes a dumping ground that decays into noise.
tier: universal
---

# Memory write

A memory file is a promise to a future-you (or a future agent) that the contents are still useful when read cold. Most agent-written memory entries break that promise within two weeks: they capture in-progress state that's stale by the next session, they re-state code patterns that the codebase already documents, or they describe a fix instead of the pattern that caused the bug.

This skill is the discipline. Four types, clear scope per type, hard NOT-list, structured body.

## When to apply

Save a memory when:
- You learn a fact about the operator's role, preferences, or working style that would change how a future session begins.
- The operator corrects your approach, or confirms a non-obvious approach, with reasoning that generalises.
- A non-obvious project decision is made and the rationale is not obvious from the code.
- An external system has a structure or quirk that took you real time to discover.

Always save (do not skip) when:
- The operator says "remember this" or "save this to memory."
- You're about to repeat advice the operator already gave you in a prior session.
- A correction surfaced a class of mistake, not a single instance.

Don't save when:
- The fact is derivable from reading the codebase, git log, or strategy docs.
- The fact is ephemeral session state (in-progress task, today's outputs).
- The fact is already in another memory file — update that one instead of duplicating.

## The four types

Every memory entry belongs to exactly one type. Mismatch is a smell — it usually means the entry is too broad and should split.

### user
Information about the operator: role, expertise, responsibilities, working style, communication preferences.

- **When to write:** Whenever you learn how the operator wants to work or what their context is. "I'm a non-technical CEO." "I prefer terse responses with no trailing summaries." "I have ten years of experience in language X but this is my first React codebase."
- **How to use:** Tailor every interaction. Adjust the level of explanation, the framing, the delivery shape.

### feedback
Guidance the operator has given about how to approach work — both corrections and confirmations.

- **When to write:** Any explicit "don't do X" or "always do Y." Critically, also when the operator confirms a non-obvious choice you made ("yes, that bundled commit was right"). Confirmations are quieter and easier to miss.
- **How to use:** Future you should not need to relearn the same correction or hesitate before applying the same confirmed approach.
- **Body shape:** lead with the rule, then a `**Why:**` line (the reason or incident), then a `**How to apply:**` line (when this kicks in).

### project
Information about ongoing work, goals, decisions, and constraints that the code does not document.

- **When to write:** When you learn the why behind a decision, the deadline behind a priority, the stakeholder behind a constraint. State that decays — these need updating as the project moves.
- **How to use:** Inform future suggestions and avoid pulling against priorities you can't see in the code.
- **Body shape:** lead with the fact, then `**Why:**` (motivation), then `**How to apply:**` (how this shapes future suggestions).
- **Watch out for:** Project memories decay fastest. Always convert relative dates ("Thursday") to absolute dates when writing, so the entry is still interpretable in three months.

### reference
Pointers to where information lives in external systems.

- **When to write:** When you learn the canonical location of something outside the working directory. "Bugs are tracked in <system>." "The latency dashboard is at <url>."
- **How to use:** Look up rather than re-discover.

## What NOT to save

These exclusions hold even when the operator says "save it." Push back politely if the request falls into one of these:

- **Code patterns, conventions, architecture.** Read the code; it is more current than memory.
- **Git history, recent commits, who-changed-what.** `git log` and `git blame` are authoritative.
- **Debugging solutions and fix recipes.** The fix is in the code; the commit message has the rationale. What memory captures is the *pattern* that produced the bug, not the line that fixed it.
- **Anything already in a project's CLAUDE.md or AGENTS.md.** Those are loaded automatically.
- **Ephemeral session state.** In-progress tasks, today's intent, current conversation context.

The fast test: "If I read this entry cold in three months, is it still true and still useful?" If no — don't save it.

## Body structure

For `feedback` and `project` types specifically, structure the body as:

```
<rule or fact in one or two sentences>

**Why:** <the reason — often a past incident, a strong preference, or a constraint not derivable from code>

**How to apply:** <when and where this kicks in for future work>
```

Knowing the *why* lets a future agent judge edge cases, not blindly follow a rule that turns out to have a different scope than the agent assumed.

For `user` and `reference` types, prose is fine. Keep it tight.

## File and index conventions

A memory entry is two artefacts:
1. **A file.** One memory per file, named for its content (`feedback_no_streak_mechanics.md`, `project_phase_2_gate.md`, `reference_analytics_dashboard.md`). Frontmatter declares `name`, `description`, and `type`.
2. **An index entry.** A single line in the project's `MEMORY.md` index pointing at the file.

The index is loaded into every session; the files are loaded on demand. So:
- Index entries should be ≤200 characters per line, organised by topic, no entry longer than a one-line hook.
- Files can be longer if the content warrants — but the entry is one *concept* per file, not a topic survey.

If `MEMORY.md` is approaching a size limit, it's a sign that index entries have grown beyond one-line hooks. Trim the index, not the files.

## Update vs duplicate

Before writing a new entry, search the existing index for related entries. The right move is often to update an existing file, not write a new one. Duplicates create:
- Drift, when one copy is updated and the other is not.
- Noise, when both surface in a search and the agent has to reconcile.
- Index bloat, which forces aggressive trimming of useful entries.

Update workflow:
1. Read the existing entry.
2. Decide: is the new fact a refinement of the same concept, or a different concept that overlaps?
3. Refinement → edit the file in place; bump or replace `description`.
4. Different concept → write a new file; update the index.

## Retire stale entries

Memory entries that have outlived their usefulness should be deleted, not left in place "in case." A wrong memory is worse than no memory because it gets read confidently.

Common retirement signals:
- The fact pointed at a system that no longer exists.
- The decision was reversed by a later decision.
- The pattern was generalised into a CLAUDE.md rule (the constitution now covers it).
- The reference URL or path moved and was not updated.

When retiring, also remove the index entry. Orphan index pointers are confusing.

## Anti-patterns to refuse

- **Writing memory as a session journal.** "Today we did X, Y, Z." Memory is not a log. Use the changelog.
- **Writing fixes instead of patterns.** "Bug X was fixed by changing line Y in file Z." This is a commit message, not memory.
- **Writing 'temporary' memory.** Memory is the long-term store. If the fact will be wrong next week, don't save it.
- **Writing without a why.** Especially for feedback type — a rule without reasoning becomes cargo culture in three sessions.
- **Saving operator anger or frustration as a memory.** Save the lesson, not the temperature.

## Calibration signal

A healthy memory directory:
- Files referenced from multiple sessions over time (you can tell by reading recent transcripts).
- Index entries that consistently come up as "load this" matches.
- Few entries needing retirement — meaning they were written tightly enough to outlast the moment.

An unhealthy memory directory:
- Entries that nobody loads even when relevant.
- Multiple files for the same concept.
- Index growing faster than the project.
- Entries that, when read cold, you can't tell why they were saved.

The cure for both shapes is the same: write fewer, write tighter, retire faster.

## Pairs with

- `decision-memo` — non-trivial decisions go through the memo first; the memo's outcome may produce a memory entry capturing the project rationale.
- `hard-evidence-under-pushback` — when a contested-fact loop resolves with the agent being wrong, the corrected fact becomes a memory update, not just a session apology.
- `vocabulary-check` — vocabulary discipline updates often produce memory entries (`feedback_vocabulary_<topic>.md`).
