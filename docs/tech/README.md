# Technical Design

One document per module, written from `_template.md` by the `tech-lead` agent, after the module's PRD is *Approved* and the UI design is approved. This document answers "which architecture"; breaking it into small tasks belongs to the implementation plan (`docs/superpowers/plans/`).

## Process

1. **Draft**: `tech-lead` writes from the PRD, the UI design and existing code; it returns open questions, questions for the PRD and decisions to record.
2. **Business questions** go back to `product-manager` to edit the PRD, not decided here.
3. **Security review**: sensitive modules (identity, tokens, cookies, sessions, private links, public keys, anything exposed anonymously) go through `appsec` before approval.
4. **Approval**: when no open questions remain `tech-lead` sets the status to *In review*; the Tech lead approves (plus the Security owner for security-critical modules); the decisions in §12 are approved with the document, no separate approval. The main session commits the approved version to the module's docs branch, in the same draft PR as the PRD (`docs/process/lifecycle.md` § 5. Committing Documents). Doc fixes caused by a major deviation during implementation (`docs/process/lifecycle.md` § 4. Design Deviations During Implementation) go in the same PR as the code.
5. **Implementation plan**: `writing-plans` breaks it into tasks, referring to the PRD's FR/AC codes and the sections of this document.

Order exception: a sensitive module may write its tech doc in parallel with UI design when the PO and Tech lead allow it and the scope of the exception is recorded in the module's document.

Any change to an approved document returns it to *In review*.

## Known constraints

`tech-lead` checks the code and `.claude/rules/` at the start of each module and records constraints and how they are resolved in the module's document.

<!-- TODO(stack): constraints inherited from the chosen stack or existing platform (e.g. framework limits, legacy auth, hosting restrictions) -->

| Item | Current constraint | Why it conflicts | Module |
|---|---|---|---|

## Catalogue

No module tech docs yet.

| Code | Module | PRD | Status | Version | appsec |
|---|---|---|---|---|---|
