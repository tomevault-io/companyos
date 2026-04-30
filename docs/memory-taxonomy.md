# Memory taxonomy — four types

Memory is the persistent layer an AI agent carries across sessions. Without discipline, it becomes a dumping ground that decays into noise. With discipline, it's the highest-leverage thing a long-running agent does.

CompanyOS uses four types. Every memory entry belongs to exactly one. Mismatch is a smell — usually means the entry is too broad and should split.

## The four types

### user

Information about the operator: role, expertise, responsibilities, working style, communication preferences.

**When to write:**
- "I'm a non-technical CEO."
- "I prefer terse responses with no trailing summaries."
- "I have ten years of experience in language X but this is my first React codebase."

**How to use:** Tailor every interaction. Adjust level of explanation, framing, delivery shape.

### feedback

Guidance the operator has given about how to approach work — both corrections and confirmations.

**When to write:**
- Any explicit "don't do X" or "always do Y."
- Critically, also when the operator confirms a non-obvious choice you made ("yes, that bundled commit was right"). Confirmations are quieter than corrections and easier to miss.

**How to use:** Future-you should not need to relearn the same correction or hesitate before applying the same confirmed approach.

**Body shape:** lead with the rule, then `**Why:**` (the reason or incident), then `**How to apply:**` (when this kicks in).

### project

Information about ongoing work, goals, decisions, and constraints that the code does not document.

**When to write:**
- The why behind a decision.
- The deadline behind a priority.
- The stakeholder behind a constraint.

**How to use:** Inform future suggestions and avoid pulling against priorities you can't see in the code.

**Body shape:** lead with the fact, then `**Why:**` (motivation), then `**How to apply:**` (how this shapes future suggestions).

**Watch out for:** Project memories decay fastest. Always convert relative dates ("Thursday") to absolute dates when writing, so the entry is still interpretable in three months.

### reference

Pointers to where information lives in external systems.

**When to write:**
- "Bugs are tracked in <system>."
- "The latency dashboard is at <url>."
- "<external API> has a quirk where Y."

**How to use:** Look up rather than re-discover.

## What NOT to save

These exclusions hold even when the operator says "save it." Push back politely if the request falls into one of these:

- **Code patterns, conventions, architecture.** Read the code; it is more current than memory.
- **Git history, recent commits, who-changed-what.** `git log` and `git blame` are authoritative.
- **Debugging solutions and fix recipes.** The fix is in the code; the commit message has the rationale. What memory captures is the *pattern* that produced the bug, not the line that fixed it.
- **Anything already in a project's CLAUDE.md, AGENTS.md, or constitutions.** Those are loaded automatically.
- **Ephemeral session state.** In-progress tasks, today's intent, current conversation context.

The fast test: *"If I read this entry cold in three months, is it still true and still useful?"* If no — don't save it.

## File and index conventions

A memory entry is two artefacts:

1. **A file.** One memory per file, named for its content. Frontmatter declares `name`, `description`, and `type`.
2. **An index entry.** A single line in the project's `MEMORY.md` index pointing at the file.

The index loads at session start; files load on demand. So:
- Index entries should be ≤200 characters per line, organised by topic, no entry longer than a one-line hook.
- Files can be longer if content warrants — but each entry is one *concept* per file, not a topic survey.

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
- The pattern was generalised into a constitutional rule (the constitution now covers it).
- The reference URL or path moved and was not updated.

When retiring, also remove the index entry. Orphan index pointers are confusing.

## Why this is operator-private

The four-type taxonomy itself is portable (and shipped publicly in this Tome). The *contents* of any company's memory directory stay private. Reasons:

- **Operator profile.** "User" entries describe a real person. Privacy.
- **Strategic positioning.** "Project" entries often capture the *why* behind decisions, including competitive analysis, vendor evaluations, hiring signals.
- **Operational tells.** "Reference" entries describe internal infrastructure. Sharing them gives operational footprint to anyone who reads.
- **Communication history.** "Feedback" entries record what the operator wanted you to do or not do; they're the closest thing to a private working relationship.

The discipline is portable; the implementation isn't. Same shape as the rest of CompanyOS.

## Calibration signals

Healthy memory directory:
- Files referenced from multiple sessions over time.
- Index entries that consistently match "load this" prompts.
- Few entries needing retirement — meaning they were written tightly enough to outlast the moment.

Unhealthy memory directory:
- Entries that nobody loads even when relevant.
- Multiple files for the same concept.
- Index growing faster than the project.
- Entries that, when read cold, you can't tell why they were saved.

The cure for both shapes is the same: write fewer, write tighter, retire faster.
