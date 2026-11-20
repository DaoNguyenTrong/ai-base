# Process Decisions

Log of AI-BASE process decisions, newest on top. The rules currently in force live in `lifecycle.md`; this file records why they changed.

### 2026-11-20 - Keep `main` as the GitHub default branch

The default branch is what a visitor, "Use this template" and a fresh clone see, so it holds the released state. Cost accepted: GitHub's compare button, a bare `gh pr create` and `isolation: "worktree"` all start from `main`, so PRs always pass `--base dev` and a worktree agent is handed its branch explicitly.

### 2026-11-16 - Reconcile `main` into `dev` through a PR (`devWrites: "pr"`)

DAONT-BASE pushes the post-release `main → dev` merge straight to `dev` because a single maintainer has no branch protection. A team protects `dev` (required PR, required checks), so a direct push is refused by GitHub and by the Bash guard. The reconcile and quick release's CHANGELOG become PRs that the Release manager merges. Cost accepted: one extra PR per release, sometimes with an empty diff.

### 2026-11-16 - Adopt the workflow from DAONT-BASE v1.6.5

Reuse a proven process (lifecycle, agents, rules, skills, command guard, risk tiers, document gates and release flow) instead of designing one from scratch. Scope changed from the source: AI-BASE is a stack-agnostic template with no product code, it adds team mode (human roles, CODEOWNERS, shared merge queue; `workflow-setup-guide.md` §15), everything is in English, `qa-frontend` becomes `qa-app`, and GitLab support is dropped. Only mechanisms and templates are carried over; no source history, approvals or metrics.

Rejected alternative: copy only `AGENTS.md` or `.claude/` (without the scripts and generators the process cannot be enforced).
