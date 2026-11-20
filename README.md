# AI-BASE

A stack-agnostic workflow template for teams building software with Claude Code (and optionally Codex). It carries the process, not a product: a lifecycle from idea to release, eleven role-based agents, approval gates, git guards, check scripts, CI for those scripts, and release configuration. It is ported from DAONT-BASE v1.6.5 (`docs/process/decisions.md`).

## What is inside

| Layer | Files |
| --- | --- |
| Process index and lifecycle | `AGENTS.md`, `docs/process/lifecycle.md`, `docs/process/{decisions,progress,system-context}.md` |
| Product documents | `docs/prd/` (personas, PRD templates), `docs/tech/` (technical design template), `docs/superpowers/plans/`, `docs/feature-map.md`, `docs/agent-error-log.md` |
| Role agents | `.claude/agents/*.md` — product-manager, business-analyst, solution-architect, ui-designer, tech-lead, implementer, qa-app, sdet, code-reviewer, pr-reviewer, appsec |
| Path rules | `.claude/rules/*.md` (stack parts are `TODO(stack)`) |
| Commands and skills | `.claude/commands/git-accept.md`; plugins `superpowers` and `release-kit` |
| Guards | `.claude/settings.json` + `scripts/claude-bash-guard.ts`, `lefthook.yml`, commitlint, secretlint, branch protection |
| Check scripts | `scripts/risk-tier.ts`, `check-comments.ts`, `check-personas.ts`, `check-prd-approval.ts`, `test-gate.sh` (+ tests) |
| CI and release | `.github/workflows/{ci,test,release}.yml`, `.claude/release-kit.json`, `scripts/setup-branch-protection.sh` |
| Codex layer | `.codex/agents/*.toml`, `.codex/hooks.json`, `.agents/skills/` |

## Quick start

```bash
bun install            # installs git hooks
bun test scripts/      # check scripts and guard
scripts/test-gate.sh   # local gate (same as CI)
```

Tool setup for each team member: [`AI_TOOLS_SETUP.md`](AI_TOOLS_SETUP.md). Branches, PRs and releases: [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Adopting the template

1. Copy the repo (GitHub "Use this template", or a clean `git archive` of a tag) and rename `AI-BASE` (`grep -rn "AI-BASE" .`).
2. Fill every `TODO(stack)` and `TODO(team)` marker: `grep -rn "TODO(stack)\|TODO(team)" --exclude=workflow-setup-guide.md .`. The order that works: `AGENTS.md` § Repository Layout → `.claude/rules/commands.md`, `testing.md`, `architecture.md` (with an architecture test) → `scripts/test-gate.sh` and `.github/workflows/test.yml` together → `scripts/risk-tier.ts` lists → `scripts/check-comments.ts` `GENERATED` → agent files.
3. Add an `AGENTS.md` inside each source directory listing the rule files that apply to it.
4. Run the repository setup in `CONTRIBUTING.md` § Repository Setup.
5. Verify the symbol tools on your stack (`.claude/rules/serena.md`).
6. Open Claude Code at the repo root and start with:

```text
Read AGENTS.md and docs/process/lifecycle.md. Check that every `file § section` reference resolves,
list anything that still names the old project or an old stack command, then start stage 0:
interview me to build the personas.
```

`workflow-setup-guide.md` (Vietnamese) is the original setup guide this template was built from.
