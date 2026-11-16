# Decision Log

Record only **platform / cross-cutting** decisions that no single module owns — infrastructure, deployment, shared auth mechanics, build and test tooling, codegen, layer boundaries — and that affect system design: architecture, module/layer boundaries, API or data contracts, security posture, or other cross-cutting structural choices with a real trade-off, external constraint, or rejected alternative that would be costly to re-derive later. Out of scope: decisions owned by one module (they live in §12 of its `docs/tech/` document), working-process decisions (`docs/process/decisions.md`), visual/branding/UI styling, copy/content wording, and one-off bug fixes — even ones that took real investigation — unless the fix itself changed a structural/contract/security decision.
Newest entry at top — prepend new entries directly below this line, do not append at the bottom.
Title format: `### YYYY-MM-DD — <decision>` (date the decision was made, not a due date).
Write only the **why** — the reasoning and rejected alternative. Never "what was done" or "how" (git log/diff already has that; do not restate file names, method names, or a narrative of the change). Keep each entry under ~80 words (excluding heading).
**Language:** write every entry in English, even when the conversation that produced it is in another language.
**Who writes:** whoever makes the change, in the same PR as the change; `pr-reviewer` checks the entry. No separate approval.
**Append-only:** never edit or delete an existing entry. To reverse one, add a new entry with a `Supersedes: <old title>` line — such an entry needs the Tech lead's explicit approval (CODEOWNERS of `.claude/`).

---

### 2026-11-16 — Gates are enforced by GitHub branch protection and CODEOWNERS, not only by agents

AI-BASE targets teams, so a client-side gate (`pr-reviewer`, `/git-accept`) is not enough: any member can bypass their own session. Branch protection on `main` and `dev` (required PR, required checks, no force push, no deletion) plus CODEOWNERS on human-gated areas makes the server refuse what the agents refuse. Rejected: agent-only gates as in DAONT-BASE, which assume a single maintainer.
