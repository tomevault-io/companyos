---
name: vocabulary-check
description: Every external string (email, landing copy, README, CLI output, dashboard label, marketplace description) passes through the project's vocabulary discipline before ship. Catches drift that compounds — internal jargon leaks into user copy, banned substitutes creep back in, the user-facing triplet stops being said. Pattern + structure for any project; the project owns its specific vocab.
tier: universal
---

# Vocabulary check

Marketing drift is silent. Every internal name, banned substitute, and pet phrase that leaks into user copy nudges the project's positioning a millimetre off-axis. Five years of millimetres compounds into "what is this product, exactly?" The cure is a literal pre-ship pass against a written list.

This skill describes the *pattern*. The list lives in your project — typically a JSON or YAML file under `.context/` or `briefings/`, or in a constitutional document.

## When to apply

Mandatory before:
- Sending any email to recipients outside the team.
- Publishing any landing page, marketing post, or README change.
- Posting to social platforms (Reddit, HN, X, LinkedIn).
- Submitting to a marketplace or directory listing.
- Creating any badge, OG card, or transactional template.
- Drafting an outbound message in a community or comment thread.

Optional but recommended:
- Internal tickets, briefings, and decision memos. Internal drift compounds into external drift over months. The earlier the discipline kicks in, the cheaper.

Skip:
- Pure agent-to-agent system text that no human will ever see.

## The vocabulary file shape

A project should declare its vocabulary in a single file with three sections:

```yaml
# .context/vocabulary.yaml (example structure — your project owns the contents)

owned_terms:
  # Words your project has staked a claim to. Use verbatim; do not paraphrase.
  - term: "<your category name>"
    note: "use as a noun, never a verb"
  - term: "<your product unit>"
    note: "always capitalised; singular outside of headlines"

required_triplets:
  # Phrases that must appear together in any hero/landing context.
  hero_triplet:
    - "<term A>"
    - "<term B>"
    - "<term C>"
    note: "all three appear in every external hero — never just one or two"

banned_substitutes:
  # Words the project has explicitly chosen NOT to use, with the reason.
  - banned: "<word>"
    reason: "implies <wrong frame>"
    alternative: "<the owned term>"

formatting_rules:
  # Mechanical rules a script or pre-commit hook can enforce.
  - "no em-dashes (U+2014) in any recipient-visible string"
  - "no all-caps for emphasis"
  - "exactly one body URL plus one footer URL in outreach emails"
  - "<your category> never appears in conversion lists"  # platform-spec rules
```

Keep this file under version control. Updates are a strategic decision, not a copyedit pass.

## The pre-ship sequence

1. **Identify what is recipient-visible.** Anything a user might see — including modal copy, toast text, error messages, badge alt text, OG image labels.
2. **Read the vocabulary file.** Recently. Don't trust your in-session memory of it; vocabulary lists evolve.
3. **Apply each rule.** Three passes:
   - Owned terms — used verbatim, not paraphrased.
   - Required triplets — present in any heroic context.
   - Banned substitutes — absent.
   - Formatting rules — mechanical (em-dash, URL count, caps).
4. **Show the diff.** When making a vocabulary-aware edit, show what you changed and which rule it satisfies.
5. **If a banned substitute appears in justified context** (e.g. quoting a competitor, citing a source), call it out so the operator can choose to keep or replace.

## Common drift patterns

These slip in repeatedly across projects. Watch for them:

- **Generic substitution by autocomplete.** "Marketplace" replaces your specific term because the model defaults to it. Re-check.
- **Paraphrase of an owned term.** "The bundle of files" instead of "<owned bundle term>". Paraphrasing dilutes ownership.
- **Mid-sentence demotion.** Hero says owned-term; subsequent paragraphs use the generic. The hero is undone if the body uses the substitute.
- **Internal name in user copy.** Internal codename for a feature leaks into the launch announcement. Audit for any term that exists in your codebase but is not in `owned_terms`.
- **Drift in error messages and toasts.** These are the most-skipped pre-ship surface and the most-seen runtime surface.
- **Formatting infections.** A copy-paste from a doc tool brings a stray em-dash, a smart-quote, or a non-breaking space.

## Mechanical enforcement

A vocabulary file is most useful when paired with at least one mechanical check:

```python
# scripts/vocabulary_audit.py — example shape

import re
import yaml
from pathlib import Path

VOCAB = yaml.safe_load(Path(".context/vocabulary.yaml").read_text())

def audit(text: str) -> list[str]:
    violations = []
    # banned substitutes
    for entry in VOCAB.get("banned_substitutes", []):
        if re.search(rf'\b{re.escape(entry["banned"])}\b', text, re.IGNORECASE):
            violations.append(f'banned: "{entry["banned"]}" — use "{entry["alternative"]}"')
    # formatting rules
    if "—" in text:  # em-dash
        violations.append("em-dash present (U+2014)")
    return violations
```

Wire this into:
- A pre-send gate on outreach (fail-closed if violations exist).
- A pre-commit hook on `*.md` files (warn-only initially; promote to fail later).
- A test that lints fixtures — the audit script is itself audited.

## Anti-patterns to refuse

- **Inventing vocabulary in the moment.** If a banned substitute appears, do not re-invent the owned term from memory. Read the file. Memory drifts; the file is canon.
- **Localising rules to one piece of content.** Vocabulary discipline is project-wide; a one-off exception in this email becomes a permanent exception by attrition.
- **Skipping the check because "the operator will catch it."** The operator's role is judgment, not pattern-matching.
- **Treating internal docs as exempt.** Internal drift compounds.

## Pairs with

- `decision-memo` — when proposing a vocabulary update, write a memo explaining what changed and why.
- `pre-ship-adversary` — vocabulary check is one of the attacks the adversarial pass should run.
- Any outreach pipeline — the audit script is fail-closed at send time.
