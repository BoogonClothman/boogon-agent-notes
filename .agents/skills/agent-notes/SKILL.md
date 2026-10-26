---
name: agent-notes
description: Use when a repository keeps — or needs to start keeping — durable decision records, such as deciding whether a change needs an Agent Note, writing or updating one, moving it between lifecycle folders, superseding or consolidating notes, or auditing, deleting, and archiving the note corpus. Applies the Agent Notes procedure in any project, independent of language or toolchain.
---

# Agent Notes

An Agent Note records a decision this repository made: the problem, what was decided, what lost and why, and what the decision cost. It carries the reasoning that code, tests, and reference documentation cannot — the part a future maintainer would otherwise reconstruct or re-litigate.

This skill is the procedure an agent follows. The repository's `.agents/notes/README.md` is the contract; where the two disagree, the contract wins for that repository. Read it before touching the tree.

## The process

A change arrives → **decide** whether it needs a note → **sweep** for the note that already owns the decision → **place** it (lifecycle × class) → **draft** it to the contract → **move or supersede** when the decision changes state → **audit, delete, or archive** when it retires → **validate and report**.

Steps 1–2 decide whether writing is justified at all. Do not skip the sweep to get to the drafting; a duplicate note is worse than no note, because it splits ownership of one decision across two files.

## Stop conditions

Stop and ask, rather than inventing a rule, when:

- `.agents/notes/` exists but its contract is missing, or contradicts this skill;
- the decision seems to need a note but no class fits;
- the change may fully or partly supersede an active note you cannot classify;
- the change would edit, move, or delete a sealed archived artifact;
- the decision has not actually been made — a `proposed/` note is appropriate, not an `implemented/` one.

## Enter the repository

1. Locate `.agents/notes/`. Read the contract (`README.md`), the directory instructions (`AGENTS.md`), and the lifecycle instructions for any folder you will touch (`implemented/AGENTS.md`, `archived/AGENTS.md`). Archived notes are read-only.
2. Read the notes around the decision. Browsing the lifecycle/class tree and searching the repository is the index; there is no `INDEX.md`, and adding one violates the contract.
3. Find the check. Repository documentation, CI, or package scripts may define a command; run the documented command rather than assuming one. If the repository has no check, this skill's [validation](#7-validate-and-report) is the check.
4. If `.agents/notes/` does not exist, do not create `docs/rfc/`, `docs/adr/`, `docs/decisions/`, or a second decision-record tree. Adoption is its own change:
   - copy this skill's `references/contract.md` (the file beside this `SKILL.md`) to `.agents/notes/README.md`;
   - create `proposed/`, `implemented/`, and `rejected/`, each with the six fixed class folders, plus `archived/`;
   - add `archived/AGENTS.md` and an empty `archived/manifest.json` (`{"version":1,"files":{}}`);
   - add a short `.agents/notes/AGENTS.md` pointing at the contract and stating that archived notes are frozen;
   - if the project has a repository-check mechanism, wire the contract's mechanical checks into it; otherwise state in the adoption note that verification is manual;
   - record the adoption itself as the first note, in class `process`.
   Ask before folding that process change into an unrelated feature.

The fixed layout:

```text
.agents/notes/
  README.md                  the contract
  AGENTS.md                  directory instructions
  proposed/<class>/          not built, or only partly built
  implemented/<class>/       shipped; facts kept current
  rejected/<class>/          declined; only while it prevents a tempting mistake
  archived/
    AGENTS.md
    manifest.json            append-only seal
    <class>/                 frozen historical notes
```

## 1. Decide whether the change needs a note

Write a note when the change leaves lasting rationale that code, tests, and existing documentation cannot carry:

- why this shape instead of the obvious alternative;
- what was knowingly given up;
- an ownership, security, compatibility, or durability boundary;
- a condition under which the decision should be revisited or a capability reintroduced;
- a rejected option that will look attractive again.

Do not write a note for:

- mechanical or local edits, including local UI presentation and interaction changes;
- facts the code, tests, or reference documentation already state;
- task plans, acceptance checklists, or implementation walkthroughs;
- change history — git owns that;
- a decision an existing note already owns.

Ask before drafting:

1. Six months from now, without this record, would a competent maintainer reverse the decision by accident or re-propose the rejected option?
2. Can the code, tests, or docs answer "why" without the note?
3. Is the knowledge durable, or does it expire with this diff?
4. Is there a real alternative that lost?
5. Is there a boundary, guarantee, or reintroduction condition someone must not violate?

Any "yes" to 1, 4, or 5 supports writing. A "yes" to 2 or a "no" to 3 argues against it. If the change is already made and nothing fits, no note is warranted; if the decision is still open, a `proposed/` note may record the open question honestly.

The output of this step is one of: **no note**, **update the owner**, **new `proposed/` note**, **new `implemented/` note**.

Resist both failure modes: a note for every pull request (documentation slop that no one maintains) and no note for a contested decision (rationale loss that the next maintainer pays for). The note is justified only when it is the one place the rationale lives.

## 2. Sweep for the existing owner

Before drafting, search the active tree for notes covering the same decision, mechanism, or rejected alternative. Search by mechanism name, symbol, path, and the words a future author would use. Read the candidates, not just their titles.

Classify what you find:

- **Full supersession** — the new note owns every unique proposition of the old one. The old implemented note may be archived in the same change if it meets the archive test; it may be deleted only through the consolidation rule below.
- **Partial supersession** — the new note owns part of the decision, or the old note still carries independent rationale. Keep both active, cross-link them, and update every fact in the old note that is still current.
- **Independent** — no relationship; proceed.
- **Obsolete proposal** — a `proposed/` note no longer worth pursuing. Reject it with an honest reason; never archive a proposal.
- **Obsolete rejection** — a `rejected/` note that no longer prevents a plausible mistake. Delete the complete set and repair its inbound links.

Record the sweep result in the change description. Do not defer a known overlap to a later corpus audit; the author with the freshest context is the one who can classify it correctly.

### Consolidation rule

An implemented note may be consolidated into its full successor and deleted only when the successor preserves every unique rationale, alternative, consequence, required verification, and named coverage gap; every inbound link is repaired; and the old note's translated counterpart and pairing record are removed in the same change. Partial supersession does not qualify. Never rewrite the old note into its opposite, and never rely on git history as the only copy of rationale.

A feature-addition note may be consolidated into a later removal note only when the feature is absent from production code, configuration, schemas, durable and wire formats, migration, and compatibility behavior; no current documentation presents it as available; and no test exercises it as supported. The removal note preserves the original motivation, why it no longer justified the feature, alternatives to full removal, the capability given up, conditions for reintroduction, and the verification that the feature is gone.

## 3. Place the note

Lifecycle is the note's status, and the note moves when the status changes:

| Lifecycle | Use when |
|---|---|
| `proposed/` | The decision is not built, or only partly built. Plans, migration steps, and open questions belong here. |
| `implemented/` | The decision shipped. The note describes shipped reality and is kept current with it. |
| `rejected/` | The proposal was declined. Keep it only while its rationale prevents a tempting, meaningful mistake. |

Class is the kind of decision. The set is fixed and may not be extended by a folder, a setting, or a project convention:

| Class | Covers |
|---|---|
| `feature` | A new user- or caller-facing capability. |
| `bug-fix` | Corrects a defect or closes a gap a postmortem surfaced. |
| `simplification` | Removes code, behavior, or surface area without adding a capability. |
| `architecture` | A structural decision about the shipped source — how packages relate, what the runtime vocabulary is. |
| `process` | Tooling, policy, or workflow around the code — checks, the package manager, vendoring — not runtime behavior. |
| `testing` | Test infrastructure and strategy. |

`architecture` is about the source we ship; `process` is about the tooling and workflow around it. `refactor` is deliberately absent: it overlaps `simplification`, whose discriminator — does observable behavior change? — already covers it. If no class fits, the note is premature or should be split; do not create a seventh folder.

Name the file `yyyy-mm-dd-topic-title.md`. The date is when the topic was first proposed — for a new note, the date of the change that introduces it; from git history once merged — not the date the note moved, was rewritten, or was archived. Use kebab-case ASCII slugs so paths and links stay stable.

One note records one decision. Two independent decisions are two notes that link to each other. Cross-references use relative Markdown links (`[topic](../../implemented/architecture/2026-…-….md)`), never bare prose or issue numbers, so they remain checkable and survive moves. Do not add `INDEX.md`.

## 4. Draft to the contract

### Header

The first three lines are exactly:

```markdown
# Agent Note: <title>

Status: <status>
```

The status agrees with the folder:

- `Status: proposed`
- `Status: implemented`
- `Status: rejected — <why, in one line>`

The status carries no date and no parenthetical. The filename holds the first-proposed date; git holds everything else. The rejection reason is the one status with content, because the verdict is what readers come for. The `Status:` line appears once, on line 3; a second one anywhere outside a code fence is a contract violation.

### Body

The first section is `## Problem`, written so it stands without the solution. Then:

**`proposed/`**

```markdown
## Problem
## Proposal
…bespoke sections…
## Alternatives considered
## Acceptance criteria
## Risks
```

**`implemented/`**

```markdown
## Problem
## Decision
…bespoke sections…
## Alternatives considered
## Consequences
```

**`rejected/`**

The proposal, frozen: keep the sections it had, including proposal-era ones. Only the header, the `## Problem` opener, `## Proposal`, and `## Alternatives considered` are mandatory; the verdict lives on the `Status:` line.

`## Decision` states shipped reality in the present tense. Proposal-era headings are spec-speak in an implemented note and the check rejects them: `## Proposal`, `## Plan`, `## Migration plan`, `## Acceptance criteria`. `## Testing`, `## Deferred`, and `## Related` are fine where they state present-tense fact. `## Alternatives considered` is mandatory in every lifecycle.

### Alternatives considered

Record each genuine alternative and why it lost — one bold-led paragraph per alternative, or a `### Why not <X>?` subsection for a contested one. Alternatives are recorded, never invented. If an older note's alternatives are genuinely unreconstructible, say so in one honest sentence inside the section; do not fabricate a debate and do not delete the section.

A decision recorded without what it beat invites re-litigation, which is the failure Agent Notes exist to prevent.

### What does not belong

- implementation walkthroughs, test inventories, or restatements of the diff;
- status annotations ("implemented!", "future: …") — the folder carries status;
- migration plans or acceptance checklists in `implemented/`;
- duplicated facts owned by code, tests, or reference docs;
- two decisions in one note.

### Language

The note's content may be written in the project's working language. The structural tokens stay in English verbatim: `# Agent Note: `, `Status:`, `Archived:`, and the canonical section headings. If the project keeps translated counterparts, each counterpart mirrors the source section-for-section and is re-recorded together with it; a translation that does not mirror the structure is not a counterpart.

Templates for the three lifecycles are in `templates/`. A template is a starting point, not a note: replace every `{...}` placeholder before validating.

## 5. Move, update, supersede, or consolidate

Moving between lifecycle folders requires updating the `Status:` line and re-satisfying the target skeleton in the same change:

- **`proposed/` → `implemented/`**: rewrite `## Proposal` as a present-tense `## Decision`; fold `## Acceptance criteria` and `## Risks` into `## Consequences` (or into a present-tense `## Testing`/`## Verification` section); drop plans and record what shipped; keep the alternatives. Facts that changed during implementation are updated in the same pass.
- **`proposed/` → `rejected/`**: add the one-line reason to `Status:` and freeze the body; keep the proposal-era sections.
- **`implemented/` update**: update paths, names, defaults, and mechanisms in place, in the same change that alters them. Do not append change history. This is tracking where the decision lives, not rewriting the decision.
- **Decision reversal**: write a new note and cross-link the old one. Never edit a note into a different decision.
- **Full supersession**: apply the consolidation rule in step 2. Partial supersession keeps both notes active and cross-linked.

When a note moves, update its inbound links and its pairing record in the same change. The check fails the move otherwise.

## 6. Audit, delete, or archive

Audit implemented notes by future decision value, not by age or word count. Apply the exits in this order:

1. **Delete** — the note only describes a small UI adjustment or a purely mechanical change. Delete its complete set (source, translation, pairing record) and repair or remove inbound links. A local bug fix, performance change, new capability, or substantive behavior decision does not qualify because its implementation was small.
2. **Keep active** — the rationale, alternatives, negative guarantees, durable or wire semantics, ownership boundary, security rule, or reintroduction condition can still guide a future change. Length does not matter.
3. **Archive** — the decision is complete, unlikely to guide future work, but its historical rationale still warrants preservation.

`proposed/` notes are never archived or deleted: reject an obsolete proposal. `rejected/` notes are kept only while they prevent a plausible mistake; otherwise delete the complete set. Do not archive toward a quota; inspect every note in scope, classify analogous groups under one principle, and record genuinely borderline calls.

Calibrated examples are in `references/calibration.md`. Word counts there demonstrate the bar, not a threshold.

### Archive procedure

An archived note is `archived/{class}/yyyy-mm-dd-topic-title.md`; `implemented` is deliberately absent from the path because only implemented notes may enter. The archival change may only:

1. move the complete set — source, translated counterpart, pairing record;
2. insert the identical `Archived: YYYY-MM-DD` line immediately below the `Status: implemented` line in every language file; the archive date may not predate the filename date;
3. re-record the pairing hashes for the metadata-only edits;
4. repair or delete inbound links from active prose. Never verify or repair links out of the archived note;
5. seal every new artifact in `archived/manifest.json`.

Do not translate, reformat, update facts, or "tidy" title punctuation while archiving. The snapshot is what it is.

### The seal

`archived/manifest.json` is append-only:

```json
{
  "version": 1,
  "files": {
    "architecture/2026-01-01-example.md": "sha256:<hex>"
  }
}
```

Every artifact under `archived/<class>/` must have an entry whose SHA-256 matches. Existing entries may never change or disappear; new artifacts may only be appended. A trusted baseline (CI's pre-change commit) makes removal detectable. Once sealed, a note is permanently frozen: do not edit, reformat, translate, move, or delete it, and do not treat it as authority for current behavior. Active prose may link into it when it intentionally cites history.

## 7. Validate and report

Run the repository's check if it has one, then verify:

- **Structure** — the notes root contains only `README.md`, `AGENTS.md`, the three active lifecycles, and `archived/`; only the six fixed classes sit under them; every note is at `{lifecycle}/{class}/yyyy-mm-dd-topic.md`; no `INDEX.md`; no stray folders.
- **Format** — exact header; status agrees with the folder and appears once; `## Problem` first; required sections present; no proposal-era headings in `implemented/`; `## Alternatives considered` present and honest.
- **Links** — every relative link in an active note resolves. Archived outbound links are exempt.
- **Pairs** — a translated counterpart mirrors the source's heading structure, carries the same status (and, when archived, the same archive date), and its pairing record matches both sides.
- **Archive** — every artifact is sealed with a matching hash; no seal changed or disappeared; metadata is correct; the set is complete.
- **Supersession** — every new note has a recorded sweep result; no duplicate owner; every affected note is updated, cross-linked, archived, or deleted.

Fix violations instead of describing them. Never claim a check passed without running it; if no check exists, say the verification was manual and list what you inspected.

Report:

- the note or notes added, updated, moved, archived, or deleted, and why;
- the supersession sweep result — what else was inspected and what happened to it;
- every borderline keep/delete/archive call with its word count and the principle applied;
- the check command and its result, or the manual checks performed.

## Reference map

- `references/contract.md` — the full contract to adopt as `.agents/notes/README.md`.
- `references/calibration.md` — worked delete/keep/archive/reject examples.
- `templates/note-proposed.md`, `templates/note-implemented.md`, `templates/note-rejected.md` — body skeletons.

Adapted from the DeepSeek Harness Agent Notes system; the calibration examples are drawn from its corpus.
