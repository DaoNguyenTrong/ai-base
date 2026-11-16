# Module Progress

Status of each stage per module (see `lifecycle.md`). Values: `-` not started, `Draft`, `In review`, `Approved`, `In progress`, `Done`. The *Code* and *Merged* columns use `-`, `In progress`, `Done`. The *Owner* column names the orchestrator of the module or phase (`lifecycle.md` § Push and PR Permissions, team rule).

Detailed PRD status lives in `docs/prd/README.md` and tech doc status in `docs/tech/README.md`; this table only summarizes. The main session updates it (`lifecycle.md` § Module Progress).

| Module | Owner | PRD | UI | Tech | appsec | Plan | Code | Merged |
|---|---|---|---|---|---|---|---|---|

## Metrics

Recorded at the end-of-module retro (`lifecycle.md` § Recurring Errors and End-of-Module Retro). Days run from the first PRD draft to the merge of the module's last PR; the `fix` ratio is `fix` commits over all commits of the module branches; the by-tier column comes from `bun scripts/risk-tier.ts --fix-ratio`.

| Module | Days | PRs | `fix` commit ratio | `fix` by tier (0/1/2) | Questions to humans | Agent tokens (k) | Notes |
|---|---|---|---|---|---|---|---|
