# Calibration: delete, keep, or archive

Word counts are discovery aids, never criteria. The test is whether a note's rationale, alternatives, negative guarantees, ownership boundaries, security rules, or reintroduction conditions can still guide a future decision.

## Delete

- **Collapsed sidebar control rail — 533 words.** Closed, minor UI behavior. Nothing in it survives the next redesign, and no boundary or rejected architecture decision is at stake.
- **Module relocation with unchanged behavior — about 200 words.** A function moved and imports were rewired; observable behavior did not change. Purely mechanical, even though it touched many files.

Do not delete a local bug fix, performance change, new capability, or substantive behavior decision merely because its implementation was small. The delete test is *what the note knows*, not *how much code changed*.

## Keep active

- **Event-sourced session boundary — 248 words.** Foundational authority and durability boundary; other decisions depend on it. Short, but load-bearing.
- **Single configuration-home resolver — 596 words.** A cross-cutting ownership rule: every new setting must respect it. Without the note, the next feature reintroduces the ambiguity.
- **Storage directory and identity policy — 628 words.** Durable layout and identity semantics. A wrong move corrupts user data, so the rationale must stay discoverable.
- **Gate scheduling policy — 400 words.** Borderline, but still constrains how checks are scheduled and resourced. Keep while the scheduling exists.
- **Deferred multimodal content block — 334 words.** States the coordinated reintroduction condition. Keep until the capability lands, then revisit.

## Archive

Archive a substantive, complete decision whose historical rationale still has value but is unlikely to guide future work. Do not archive records that meet the delete criterion; do not use the archive as a holding pen for notes nobody wants to judge. Never archive a proposal.

A useful test: if the note disappeared tomorrow, would anyone's next decision change? If yes, keep. If no, but the history explains why the code looks the way it does, archive. If nobody would notice either way, delete.

## Rejected notes

- **Keep: merging two packages — 426 words.** The temptation remains meaningful; a future maintainer will propose the merge again, and the note explains why it lost.
- **Delete: streaming progress through tool calls — 972 words.** Its premise is obsolete; the note no longer prevents a plausible mistake. Length does not save it.
- **Delete: dropping terminal metadata — 362 words.** A later decision resolved the question, so the rejection now misleads rather than guards.

## Borderline calls

Record every genuinely borderline keep/delete/archive call in the handoff, with its word count and the principle applied. A borderline call is information, not a failure; a silent coin flip is the failure.

These examples are calibrated against the DeepSeek Harness Agent Notes corpus, the reference implementation of this system. Adapt the principles, not the verdicts: a different codebase has different load-bearing decisions.
