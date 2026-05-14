---
name: docs-write
description: ALWAYS run this skill when writing or reviewing anything under `tomevault-web/content/docs/` (the practitioner reference surface) or `briefings/*` that ships externally. Encodes the post-LLM 2026 voice discipline: bans em-dashes in reference pages, enforces sentence-case headings, kills AI scaffolding phrases, and prescribes a per-doc-type voice (reference is austere, explanation is opinionated, tutorial is task-shaped). Apply BEFORE writing, then again as a pre-ship pass. The whole point is that AI-generated docs flooded the web 2023-2025 and the corrective movement is real; failing this skill means our docs read like everyone else's and we forfeit the category-creator legitimacy that v11.1 is built on. Triggers on "write docs", "/docs/...", "MDX", "reference page", "spec page", "API docs", "/learn/...", or any frontmatter-bearing markdown file inside tomevault-web/content/.
tier: universal
last_verified: 2026-05-13 (refined with 3 lessons from the retroactive em-dash strip the same day)
sources:
  - briefings/2026-05-12-docs-implementation-plan.md
  - 2026-05-13 research pass on Stripe, Twilio, Helm, Snyk, Cargo, OpenAPI, npm, Rust Book, Terraform docs
  - Microsoft Writing Style Guide (2025 update)
  - Google Developer Documentation Style Guide
  - Diátaxis (diataxis.fr)
  - Datadog Vale wiring writeup
---

The discipline below is enforced, not aspirational. Every rule has a Vale-pack analogue listed at the end so it can run in CI. Apply this skill before writing; then again on a pre-ship pass.

## The non-negotiables

These are hard fails. A page that breaks any of them does not ship.

### Don't open pages with a blockquote callout

No `> **...` as a page-header decoration. Markdown blockquotes render in Fumadocs as an indented highlighted box that looks like a quoted pull-quote from somewhere else. Even when the content is real metadata (status, version, last-verified, prerequisites, command signatures), the visual treatment confuses the reader, who expects pull-quotes to carry voice borrowed from another source.

Page-level metadata that needs surfacing belongs in one of three places, in order of preference:

1. **The page's `description` frontmatter.** Fumadocs renders it as the subtitle directly under the title. This is the right home for "what this page is about" framing.
2. **The first prose sentence of the body.** "The `tomevault publish` command validates a Tome manifest and submits it to TomeVault." Beats `> **Command** \`tomevault publish\``.
3. **A small "Status" or "Last verified" section at page end.** Archival metadata that the reader doesn't need before reading the page.

Never use `> **...` as a page-header decoration. The lesson behind the lesson: when adapting precedent moves (in the case that produced this rule, the OpenAPI "version + date + editor" legitimacy block), check the rendered output of the technique you're using, not just the semantic intent. Markdown blockquote renders inconsistently across themes; what looks fine in one preview is a confusing indented box in another.

### Don't write for an audience that doesn't exist yet

Pre-traction, the reader is a curious stranger encountering the thing for the first time, not a member of a thriving community. That means do not ship:

- **Migration / upgrade pages** for things nobody migrated from. If our own internal v0 had three users, there is no migration audience. Write the rule, not a journey.
- **"Common mistakes" sections that fabricate failure modes** we have never actually seen. List a mistake only after a real user has made it, not from imagined symmetry with the success path.
- **"Growing community" language**. No "the community", no "users often", no "many people have asked".
- **Testimonial-shaped prose** ("teams that adopt X find Y"). Empty until someone really did.
- **Changelog of breaking changes from V0 to V0.1** when nobody was on V0.

Write what the thing IS. Write what it DOES. Show a worked example. Stop. The migration page, the FAQ, the troubleshooting guide, the version history are *earned content* — they ship when there is real adoption to migrate, real questions to answer, real bugs to document. Until then the docs say "here it is" and shut up.

Pattern test: before writing a page, ask "who is the reader and what did they do five minutes ago?" If the answer requires an audience that does not yet exist, the page is not ready to write.

### Em-dashes are banned in reference pages

No `—` in `tomevault-web/content/docs/**/spec/**` or any field-table page. Use commas, periods, or parens instead. Em-dashes are the single most reliable AI tell as of 2026; reference voice has no use for them anyway (the punctuation is for parenthetical drama, and reference pages have no drama).

In explanation and tutorial pages, em-dashes are allowed but capped at one per 1,000 words. Use them where a comma would create ambiguity, not as connective tissue.

**Pattern library for replacements.** Lifted from the 2026-05-13 retroactive strip, where the same em-dash appeared in four recurring shapes. Each shape has a default fix:

| Context | Example before | Replacement | Example after |
|---|---|---|---|
| Parenthetical (X — Y — Z) | "If you are new to the vocabulary — Tome, Skill, AGENTS.md — start at..." | parens or comma | "If you are new to the vocabulary (Tome, Skill, AGENTS.md), start at..." |
| Enumerative (X — A, B, C) | "names the Tome itself — id, version, authors" | colon | "names the Tome itself: id, version, authors" |
| Sentence connector (X — Y is...) | "Multi-author Tomes are first-class — single-author Tomes are a list of length 1" | period | "Multi-author Tomes are first-class. Single-author Tomes are a list of length 1." |
| Result-state in table cell (`error_code` — action) | `lockfile_hash_mismatch` — refuse to render | period or restructure into a separate column | `lockfile_hash_mismatch`. Refuse to render. |
| Empty placeholder in a mapping table (\| — \|) | `\| — \| manifest.authors[0].name \|` | explicit literal `(none)` / `(removed)` | `\| (none) \| manifest.authors[0].name \|` |
| Section heading (Example 1 — Foo) | `## Example 1 — Minimal config-only Tome` | colon, then sentence case | `## Example 1: minimal config-only Tome` |

**Code-comment em-dashes inside teaching examples** (`# Valid — paths resolve...` inside a YAML or shell block that the page is teaching) follow the prose rule: replace with a colon. The page is teaching the convention; the example should embody it.

**Code-comment em-dashes inside user-supplied data examples** (a real CLI output transcript, a third-party file's contents, a quoted error message) stay verbatim. Those are data we are showing, not prose we are writing. Quoting reality is fine.

### Sentence case for every heading

`### name` not `### The Name Field`. `## Worked examples` not `## Worked Examples`. Apply across H1 through H6 and across sidebar titles in `meta.json`.

### Banned-phrase canon

Replace, never use:

| Banned | Use instead |
|---|---|
| leverage | use |
| utilize | use |
| in order to | to |
| at this time | now (or delete) |
| simply, easily, quickly | delete |
| just (as scaffolding: "just run", "just edit") | delete; legitimate adverbial use ("not just a file", "not just which formats") is fine |
| please note, it's important to note | delete; lead with the note |
| let's, let's dive into, let's explore | start with the thing |
| in essence, in summary, ultimately, overall | delete |
| moreover, furthermore, consequently, notably | delete or use a period |
| there is, there are, there were | rewrite with a real verb |
| you can | imperative ("Set the value", not "You can set the value") |
| e.g. / i.e. / etc. | for example / that is / be specific |
| delve, delve into | examine, read, study |
| harness, unlock, unleash, empower, supercharge | use the literal verb |
| streamline, optimize, enhance | be specific about what changes |
| navigate (as in "navigate the landscape") | delete |
| comprehensive, seamless, robust, intuitive, powerful | delete; show, don't tell |
| innovative, revolutionary, game-changing | delete |
| crucial, essential, key (when intensifying) | delete |
| realm, landscape, tapestry, synergy, ecosystem, journey, paradigm | use the literal noun |

These are the words AI overproduces. Stripping them does not lose information; it gains clarity.

### Reference-page voice is austere

Per Diátaxis: "Describe and only describe. Adopt an austere and uncompromising tone. State facts plainly without explanation or instruction."

For a spec field, the page is: heading (field name, code-fonted) → type, required/optional, version-added → one-sentence purpose → example block → constraints (table or short list) → cross-links. That is the whole page. No marketing chrome. No "this field allows you to". No closing zoom-out.

### Length targets per page type

- **Field reference page**: 150-250 words. Over 400, split. Under 80, consolidate with a sibling.
- **Concept explanation page**: 400-900 words.
- **How-to page**: 200-500 words.
- **Tutorial page**: 600-1500 words.

### Sentence-rhythm discipline

At least one sub-15-word sentence per paragraph. Three consecutive sentences over 25 words is a fail. Paragraphs cap at 5 sentences. No three-act paragraph (setup → expansion → zoom-out close); reference pages have no zoom-out.

### Bulleted-list cap

No more than 3 consecutive bulleted lists. After 3, break the rhythm with a paragraph or a table.

### No "what you'll learn" / "what we'll cover" preambles

Lead with the thing. The reader can see the headings.

## Per-doc-type voice

| Type | Voice | Lead with | Tense | Example opener |
|---|---|---|---|---|
| Reference | Austere, declarative | The field/concept name | Present | "The `name` field identifies the Tome." |
| Explanation | Opinionated, terse | The point, not the setup | Present | "A Tome is a manifest. Three blocks plus one sibling lockfile." |
| How-to | Imperative, task-shaped | The verb | Imperative | "Validate a manifest locally." |
| Tutorial | Friendly but tight | The outcome | Future-tense outcome, imperative steps | "By the end you will have a working Tome." |

The same content rewritten across types:

**Reference (banned)**: "You can use the security block to declare permissions, attestation, and scanner ruleset. The block is optional and has a permissive default."

**Reference (correct)**: "The `security` block declares scanner ruleset, permissions, attestation hash, and scan-required flag. Omitting the block triggers the strict default."

**Explanation (correct)**: "The security block is the Snyk-shaped overlay on the Helm-shaped manifest. Manifest says what you bundle; security says what you do. Deny by default."

**How-to (correct)**: "Add a security block to your `tome.yaml`. Set `permissions.filesystem: [read]` if your Tome reads files. Set every other permission list to `[]`."

## The category-creator tactics worth stealing

These are not voice rules; they are structural moves that make a spec become a standard. Apply them where the surface supports it.

### Two-track split: normative spec next to narrative authoring guide

The spec page uses RFC-style language (MUST / SHOULD / MAY) and field-by-field tables. A sibling authoring page explains the why, the gotchas, the worked examples. Both linked from the section landing. Pattern lifted from Cargo (Book vs Reference), Helm (Charts vs Chart Best Practices), Terraform (Language vs Tutorials), OpenAPI (Spec vs Guides).

For TomeVault: `/docs/tome-yaml/spec/*` is the normative spec; a sibling `/docs/tome-yaml/authoring` page is the narrative explainer. Cross-link both at the top. Already partially structured this way; finish the authoring page.

### Version + date + named ratifier block on every spec page

Each spec page top-of-doc carries:

```
Status: tome-spec-draft-01
Ratified: pending (≥5 internal Tomes required)
Editors: Oli Boyd
```

This is the OpenAPI move. It signals "standard, not vendor doc" and is what absorbs competing specs once dated.

### Reserved-namespace clauses

Stake claim explicitly. Helm's wording is the template: "TomeVault reserves use of the top-level keys `manifest`, `includes`, `targets`, `security` and the directory `.tome/`. Authors MUST NOT use these names for other purposes."

This sentence is how a convention becomes a standard.

### Live builder embed on the spec page (Stripe move)

Right rail of `/docs/tome-yaml/spec/manifest` is a live form whose state copies out as a valid YAML block. Reader writes their manifest by reading the spec. Highest-leverage friction collapse available; needed for the category to legible.

Defer to a later phase if Studio integration is not ready. Track as a separate briefing.

### Verb-per-page for CLI docs

When `/docs/cli/*` ships: one page per command (`tomevault validate`, `tomevault publish`, etc.). Identical template per page: one-line description → exact invocation → expected output → exit codes → CI snippet (copy-pasteable GitHub Action). Lifted verbatim from Snyk and Twilio.

## Pre-ship checklist

Run on every draft. Each item is mechanical; either it passes or it does not.

1. **Em-dash count**: zero on reference pages, ≤1 per 1,000 words elsewhere.
2. **Banned-phrase grep**: no hits against the canon table.
3. **Sentence case** on every heading. No Title Case.
4. **Lead with the thing**: no "let's", no "in this section", no setup paragraph.
5. **One concept per page** for reference; one task per page for how-to.
6. **Required/optional**: stated explicitly on every spec field.
7. **One example block** per field (not zero, not three).
8. **Cross-links at the bottom** as "Next" + related. No mid-paragraph "see also".
9. **No three-act paragraphs**: no zoom-out close on reference pages.
10. **Word count within target** for the doc type (150-250 for field ref, etc.).

## Vale CI wiring

These rules become enforceable via Vale. The full pack lives at `tomevault-web/styles/TomeVault/`. To stand up:

1. `npm install -D @vvago/vale` (or use the standalone binary in CI)
2. `.vale.ini` in `tomevault-web/`:

```ini
StylesPath = styles
MinAlertLevel = suggestion

Packages = Microsoft, write-good, alex, proselint

[content/docs/**/spec/**]
BasedOnStyles = Microsoft, write-good, TomeVault, TomeVaultReference

[content/docs/**]
BasedOnStyles = Microsoft, write-good, TomeVault
```

3. `styles/TomeVault/words.yml`: the banned-phrase canon from the table above, machine-readable. Each entry has the banned word, the suggested replacement, and a one-line reason.

4. `styles/TomeVault/em-dash.yml`: `scope: text` regex `—|--` with level `error` for the spec scope and `warning` otherwise.

5. `styles/TomeVault/sentence-length.yml`: warn on sentences over 30 words.

6. `styles/TomeVaultReference/austere-voice.yml`: sentence-starter banlist scoped to reference pages only. Bans `Let's`, `In this section`, `We'll`, `You'll`, `Throughout this`, `This page covers`, `It's important`.

7. GitHub Action that runs Vale on every PR touching `content/docs/**` and posts inline comments on the diff (Datadog's pattern).

8. Local recipe in `justfile`: `just docs-check` runs Vale against staged docs files.

The check runs in CI, not in the operator's head. That is the whole point.

## How this skill improves over time

The feedback loop is concrete: this file is versioned. When the operator catches a tell that the current rules missed, the fix is:

1. Find the banned phrase or pattern in the rejected draft.
2. Append it to the appropriate Vale rule file under `styles/TomeVault/`.
3. Append a one-line rationale to this skill file, dated.

This skill stays small. New rules earn their place by catching a real failure; rules that stop firing get retired.

### Changelog

**2026-05-13 (initial)** — Skill landed. Encoded the post-LLM 2026 voice discipline, the banned-phrase canon, per-doc-type voice, length targets, the category-creator tactics, the Vale CI wiring blueprint.

**2026-05-13 (refinements after the retroactive strip)**

- Added the em-dash replacement pattern library (6 recurring shapes from the actual strip). Future authors and the Vale rule writer now have a concrete fix per context, not a generic "remove em-dashes" instruction.
- Split the `just` ban: scaffolding usage is banned, legitimate adverbial use ("not just X", "not just which Y") is fine. The original `simply, easily, quickly, just` row was too coarse and would false-positive on legitimate prose. Vale rule needs the same context discrimination when it ships.
- Clarified the code-comment carve-out: em-dashes inside teaching examples follow the prose rule (we teach the convention by embodying it); em-dashes inside quoted user-supplied data stay (we cite reality, we do not edit it).

**2026-05-13 (audience-reality rule)**

- Added the "don't write for an audience that doesn't exist yet" non-negotiable. Caught a "Migrate from tome.json" page in the Phase 1 batch that implied an external community of authors using `tome.json`. No such community exists; tome.json is an internal TomeVault backfill format. The page was hallucinating a successful-spec-evolution narrative. Deleted, with every cross-link cleaned. Pattern test added: "who is the reader and what did they do five minutes ago?"

**2026-05-13 (the em-dash reflex)**

- Confirmed pattern: every batch of fresh docs writes em-dashes back in despite the rule. The first /docs/tome-yaml pass: 73. Batch C-G: 95. Batch H: 23. Batch H+ (recipes): 12. The bulk-strip works (sed pattern from the previous changelog entry stays effective), but the discipline failure recurs at draft time. Two adjustments:
  - **Reflex training:** when typing a bullet-list `Item — description` shape, the default key should be a colon, not an em-dash. The colon reads as well or better in every case the bulk-strip handled. There is no draft-time use case for the em-dash in reference pages.
  - **Workflow rule (formalised):** after writing any batch of MDX files, run `grep -c '—' content/docs/**/*.mdx` before committing. If the count is non-zero on reference pages, run the context-aware sed pass before staging. This is the closing-bracket discipline applied to em-dashes: the check is mechanical, the cost is one bash command, the catch rate is 100%.

**2026-05-13 (no-blockquote-as-page-header)**

- Caught 43 pages opening with `> **Status** ... · **Editor** ... · **Updated** ...` style blockquotes. Intent was the OpenAPI "version + date + editor" legitimacy block; execution rendered as indented highlighted pull-quote boxes that read as out-of-place quoted text rather than metadata. Operator flagged the visual treatment immediately. Stripped every occurrence. New non-negotiable rule added: never use `> **...` as a page-header decoration. Lesson: when adapting precedent moves from outside the codebase, check the rendered output of the technique on the actual theme, not just the semantic intent. Markdown blockquote behaviour varies across renderers; what reads as metadata in OpenAPI's CSS reads as a quoted aside in Fumadocs.

**2026-05-13 (deferring rather than fabricating, applied to Studio + Relay)**

- Studio docs shipped without screenshots; Relay docs shipped without webhook payload depth. Both were flagged to the operator as "deferred until X" rather than padded with prose-only approximations. The audience-reality rule extends: when a section needs visual or implementation depth I can't produce, the page says so explicitly ("The Relay server lives in a private repository; the public documentation is the contract rather than the implementation") rather than fabricating. The "What this section doesn't cover yet" sub-section pattern is the right shape for that admission.

## What I personally got wrong on the 2026-05-13 batch (calibration)

The first Phase 0 and Phase 1 docs pass shipped with one em-dash per ~80 words on average. The catastrophic file was `spec/security.mdx` (9 em-dashes in 802 words). Bullet-list drift hit 5 consecutive in `examples.mdx`. These are exactly the failures this skill is built to prevent.

Revising those pages to match this skill is a separate task; the operator decides whether to ship the revision as a single retroactive commit or fold it into the next content batch.

## Quick test (apply to your own draft)

Before shipping a page, ask:

1. Did I use any em-dashes? Remove.
2. Did I write "this field allows you to"? Rewrite.
3. Did my last paragraph zoom out to a bigger point? Delete it.
4. Is my opening sentence the thing, or scaffolding to the thing? If scaffolding, replace with the thing.
5. Could a Vale rule catch what I just did? If yes and the rule does not exist yet, add it.

That is the skill. Five questions and a CI pack. Everything above is the implementation of those five questions.
