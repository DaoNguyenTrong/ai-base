# Software Development Lifecycle

This document describes the path from idea to release: who does each stage, who approves it, and when it is done. `AGENTS.md` holds the general rules (commits, push and PR, tools, where decisions are recorded); the detail of each stage lives here. The approver roles named below are defined in `AGENTS.md` § Team Roles.

## 1. Stages

| # | Stage | Output | Who does it | Approval gate | Done when |
|---|---|---|---|---|---|
| 0 | Idea and vision | Personas (`docs/prd/00-personas.md`), vision, module list, priority order | Main session (`superpowers:brainstorming`) with the orchestrator; personas written by `product-manager`, reviewed by `business-analyst` (§1b) | PO | Personas are *Approved*, the module catalogue in `docs/prd/README.md` is fixed |
| 1 | Requirements | PRD in `docs/prd/` | `product-manager`, reviewed by `business-analyst` | PO | PRD is *Approved* |
| 2 | UI design | Design boards on the canvas, tagged with requirement codes | `ui-designer` | PO + Design lead | Every FR with a new screen or new interaction pattern has a board; screens that follow an existing pattern only name the pattern; dark boards only for new patterns (`ui-designer.md` rule 1); approved |
| 3 | Technical design | Document in `docs/tech/` | `tech-lead` (`appsec` first for sensitive modules) | Tech lead (+ Security owner for security-critical modules) | Document is *Approved*, every FR/AC has a design section |
| 4 | Plan | Plan in `docs/superpowers/plans/` | Main session (`superpowers:writing-plans`) | Tech lead | Plan approved; the Tech lead also decides whether to grant push and PR permission |
| 5 | Implementation | Code, tests, `docs/feature-map.md` entry | `implementer`, one task at a time | By tier (§2 Risk Tiers) | All tasks of the phase done, `scripts/test-gate.sh` passes |
| 6 | Review and merge | PR merged into `dev` | `pr-reviewer` (docs-only PRs: `/git-accept`) | Automatic, except human-gated areas | PR is in `dev`; after the module's last PR the main session runs the retro (§8) |
| 7 | Release | Release, changelog | Skill `release-kit:git-release` | Release manager | Tagged and merged into `main` |
| 8 | Operation and feedback | Feedback from the product built on AI-BASE | PO | — | Back to stage 0, 1 or 5 depending on work size |

The main session is the orchestrator: it sends every question to `solution-architect` to answer on behalf of the user, asks the human approvers only the questions that agent escalates (business → PO, technical → Tech lead, security → Security owner), passes answers to agents, calls agents in order, and does not do an agent's work itself. Subagents cannot ask a human and cannot call other agents.

To cut calls, the main session batches the questions of one stage into a single `solution-architect` call. A purely technical question with a reasonable default is decided by the main session and recorded in the plan, not sent.

## 1b. Personas

A persona is a portrait of a real group of users, built from goals and behaviour, not from age or job title. A persona differs from a role: a persona says who uses the product and why, a role (PRD-00 § Roles) says who may do what. A new product starts no PRD until `docs/prd/00-personas.md` is *Approved*; `scripts/check-personas.ts` enforces this in `scripts/test-gate.sh` and CI.

**How to build them.** The main session orchestrates; the document is written from `docs/prd/_template-persona.md`, lives on the docs branch and goes through `/git-accept` like a PRD (§5).
1. **Gather evidence.** The main session interviews the PO following `superpowers:brainstorming`, asking about real people and the last time they hit the problem. Sources in order of preference: interviews with end users, metrics or tickets, domain experts, the PO's own knowledge.
2. **Cluster** by goals and behaviour: two people with the same job title but different goals are two personas. Keep 2-5 personas, choose exactly one primary persona, and at least one anti-persona (`AP-nn`).
3. **Write.** `product-manager` writes from the template; every claim is labelled `[User]`, `[Data]` or `[Assumption]`. Without research a proto-persona is allowed, but every `[Assumption]` has a way to verify it in §6 of the document.
4. **Settle conflicts** between personas: this is a business decision, so `solution-architect` escalates it to the PO, never decides it, and never invents a persona.
5. **Review.** `business-analyst` reviews against the checklist below, then the PO approves; `scripts/check-prd-approval.ts` checks this order as it does for PRDs.

**Approval checklist:** each persona has at least 2 goals, 2 pains and 2 situations; no two personas share the same main goal; exactly one primary persona and at least one anti-persona; no more than half the rows of a persona are `[Assumption]`; no features or solutions described; devices and language are clear enough for `ui-designer` to choose a layout.

**Downstream use.** Persona codes enter the traceability chain (§7):
- **Module PRD:** §3 lists the personas using the module and its primary persona; each user story in §4 carries a `P-nn` code.
- **Design:** each board names its primary persona, used to choose device and information density.
- **E2E tests:** name scenarios by persona.
- **`solution-architect`:** uses personas as the basis for answering on behalf of the user.

**Changes.** Editing an approved persona raises the version and returns the document to *In review* (§3); PRDs that have stories for the edited persona are re-examined. Stage 8 feedback is the main source for updating personas. The end-of-module retro (§8) checks whether any persona no longer has a story serving it.

## 2. Work Size

The main session sets the size at the start of each task and says so; the orchestrator may change it.

| Size | When | Path |
|---|---|---|
| **S** | Bug fix, chore, small change that alters no behaviour or contract | Skip stages 0-4. Bugs: failing test first. `implementer`, then review by tier |
| **M** | Change or add behaviour in an existing module | Edit the PRD (raise the version, back to *In review*, then re-approve), design and tech doc only for the affected parts, a short plan, then stages 5-6 |
| **L** | New module, or a change touching permissions, sessions or identity | All stages 0-6, `appsec` if sensitive |

Size S must not add user-visible behaviour or change an API. If an S task turns out to need a behaviour change, stop and raise it to M.

### Risk Tiers

Size decides which documents are needed; the tier decides how closely to check. The tier of a change is the highest tier among its files, computed by `bun scripts/risk-tier.ts --base origin/dev`, not self-assessed. The author may raise the tier, never lower it.

| Tier | Which changes | Checks |
|---|---|---|
| **0** | Docs, process text, locale files, generated code and API clients, test-only changes | CI only. The main session reads the diff itself, no review agent |
| **1** | Everything else: ordinary behaviour in non-sensitive areas (CRUD, screens, lists, filters) | `implementer` on Sonnet, tasks batched by screen. No review after each task; one Sonnet review pass for the whole phase PR (`pr-reviewer`). No mutation checks, no Opus |
| **2** | Authentication, tokens, cookies, sessions, permissions, tenant/data isolation, API keys, locks and concurrency on shared data, DB migrations, CI and hook config, dependency manifests and lockfiles, container files, `.env*`, app config, `.claude/settings*.json`, the tier script itself | Full process: `appsec` reviews the tech doc before the plan, an Opus reviewer after each task, about 5 mutation checks on tests. The final phase review is `pr-reviewer`, called by the main session with `model: opus`; there is no separate final review. The area's human approver merges it (§3 Human-gated areas) |

The path and marker lists for each tier live only in `scripts/risk-tier.ts`. The tech doc of a new module names the tier-2 paths the module adds, and the module's first PR adds them to the script.

**Main session while running a plan.** Every tool call of the main session re-reads the whole context, so its model and session length are the largest cost.
- Once the plan is approved, the main session asks the orchestrator to switch to `/model sonnet` for the run. Opus is kept for brainstorming, planning and tier-2 review.
- Each phase runs in a fresh session, continuing from the plan's ledger (`.superpowers/sdd/<plan>/progress.md`).
- The orchestrator decides; the main session does not change its own model.

## 3. Gates Between Stages

- **Do not enter a later stage until the earlier one is approved.** Exception: a sensitive module may write its tech doc in parallel with UI design when the PO and Tech lead allow it and the scope of the exception is recorded in the module's document.
- **Business questions that arise in a later stage go back to the PRD**, edited by `product-manager`, not decided where found. Technical questions belong to `tech-lead`.
- **Changes after approval:** editing a PRD returns it to *In review* and requires re-examining the affected design, tech doc and plan. Affected documents also return to *In review* and raise their version. Exception: filling or fixing the link and metadata rows (*Design*, *Plan*, *Updated*) is not a change.
- **After 3 fix rounds on a PR that still does not pass:** `pr-reviewer` returns `ESCALATE`, naming the document to rethink (PRD, design or plan). There is no 4th fix round.
- **Human-gated areas:** authentication, tokens, cookies, sessions, permissions, tenant/data isolation, API keys, `.claude/`, `.github/`, `AGENTS.md`, CI and Lefthook config, `scripts/risk-tier.ts`, DB migrations, deployment config, signing and release config, edits or deletions of existing `.claude/decisions.md` entries or a `Supersedes:` line, plus <!-- TODO(stack): product-specific sensitive areas (e.g. payment flows, encryption keys, webhook secrets) -->. `pr-reviewer` does not merge these PRs; they are merged by the human approver of the area (`AGENTS.md` § Team Roles).

## 4. Design Deviations During Implementation

When the code cannot do what the approved tech doc says:

| Level | When | Handling |
|---|---|---|
| **Minor** | Does not change the API contract, permissions, stored data model or user-visible behaviour (for example an internal class name, adding an index, splitting a service) | `implementer` continues, records it in the *Minor design deviations* section of its report and in the PR description with the reason. `pr-reviewer` blocks if the deviation is not truly minor or not recorded. The tech doc is not edited |
| **Major** | Changes any of the above | `implementer` stops the task and returns the deviation with evidence and a proposal. The main session calls `tech-lead` to edit the doc on the working branch (raise the version, back to *In review*); business questions go through `product-manager` first. The Tech lead re-approves, then `implementer` continues. The fix ships in the same PR as the code |

When unsure, treat it as major. To surface major deviations early, the plan makes the highest technical-risk part the first spike task.

## 5. Committing Documents

- **A module's PRD and tech doc** share one branch `chore/<module>-docs` cut from `dev` at the first draft, with **one draft PR** for the whole module. The main session commits each version to that branch, so version history and review reports (PR comments) are not lost between work sessions. Starting the module's PRD is the push and the opening of that draft PR.
  - Each time an approver approves a document (PRD first, then tech doc), the main session commits the approved version to the branch. The design board link goes in the PRD's *Design* row.
  - Agents in later stages (`ui-designer`, `tech-lead`, `writing-plans`) read the documents from this branch.
  - The PR is merged **once** through `/git-accept`, when every document of the module is *Approved*. If design or the tech doc is blocked for long, the PO may allow the approved PRD to merge alone.
- **The plan** is committed by the main session on its `feature/*` branch, in the same PR as the code.
- **Every PR**, including docs-only PRs, has a `## Changelog` section in the format of `release-kit:git-mr` (`- Added: ...`, `- Fixed: ...`, or just `none` for internal changes), and is always followed by another `## ` section: `git-release` reads this section up to the next `## ` heading, so a free-text trailing line (such as a source line) would break it. `git-release` builds the release CHANGELOG from these sections; do not edit `CHANGELOG.md` by hand.
- **Doc fixes caused by a major deviation** are committed by the main session on the working branch, in the same PR as the code; this way `pr-reviewer` always compares the code with the right document version.
- `implementer` does not edit or commit PRDs, tech docs or plans.

## 5b. Document Length

- **Plans** describe each task by: interface (public signatures, DTOs, endpoints), file list, test cases and acceptance criteria. Copy short code only when a signature or algorithm is hard to say in words. Do not write full code in advance: `implementer` rewrites it, and the version in the plan drifts from the real code.
- **A module PRD** usually aims at about 500 lines. This is a soft limit: a large module may exceed it if the reason is stated at the top of the PRD. Do not cut scope to keep the line count; cutting scope is a decision for the PO.
- **Short-form tech docs** are for modules marked `not needed` in the *appsec* column of `progress.md`. Required: §1, §3, §4, §5, §11, §12 and §13 of the template. The other sections may be one line, "Not applicable: <reason>". Aim at about 200 lines. For these modules `business-analyst` reviews the PRD once; a second pass only if the first has a `blocking` finding.

## 6. Push and PR Permissions

By default push or open a PR only when a human orders it. At stage 4, when approving the plan, the Tech lead may grant **one permission for the whole plan**: each time a phase is done and `scripts/test-gate.sh` passes, `implementer` may push that phase's branch and open one PR into `dev`. The permission is recorded at the top of the plan and in the brief given to `implementer`; it does not apply to `main`, and does not replace the `pr-reviewer` gate.

**One PR per phase.** Each phase of the plan is a branch `feature/<module>-phase-<n>` and one PR into `dev`. A PR aims at no more than about 1,500 lines of source, excluding tests and generated files (<!-- TODO(stack): generated paths, e.g. OpenAPI export, generated API client, lockfiles -->); a phase touching human-gated areas (§3) should be smaller. When planning, a phase estimated above this is split in two. If it grows past it during work, split at a task boundary, not inside a task. The 1,500 figure is reviewed at the retro of the first module run through this process.

**One level of stacking.** So implementation need not wait for review, the branch of phase N+1 is cut from the branch of phase N once phase N's PR has passed `pr-reviewer` (passed, or in its first fix round). Do not stack more than one level, and do not stack when phase N has had more than one fix round. When phase N merges, the N+1 branch is rebased onto `origin/dev` before its PR is opened or reviewed, so the one-PR-at-a-time rule below still holds.

**One purpose per PR.** Do not mix kinds of work in one PR, especially docs awaiting approval with tooling or config changes: a mixed PR can merge only when its slowest part is done, and must pass the strictest gate of its parts. When asked to mix, the main session states that cost first.

**Merge order: near-linear history.** At any moment only one PR into `dev` is reviewed and merged. Before review the branch must be rebased onto the latest `origin/dev` (`git rebase origin/dev`, then `git push --force-with-lease`); merge with `gh pr merge --merge` so each PR still has a merge commit marking its boundary. Rebase the next branch only after this PR is merged. A branch that is not rebased gets *not ready* from `pr-reviewer` and `/git-accept` (`NOT_READY`), which does not count as a round. The main session orders the queue and requests rebases.

**Team rule.**
- One orchestrator per module or phase at a time, recorded in the *Owner* column of `progress.md`.
- Each member works in their own checkout; two implementers never share one.
- One PR into `dev` at a time through a shared queue: the GitHub merge queue once there are 3 or more PRs per day into `dev` (`CONTRIBUTING.md` § Repository Setup), otherwise an agreed convention of who rebases and merges first. An unrebased branch makes `pr-reviewer` return `NOT_READY`.
- `progress.md`, `docs/prd/README.md` and `docs/tech/README.md` often conflict when several people edit them: edit only your module's row, and rebase right before merging.

**Branch cleanup after merge.** Right after merging, the merger (`pr-reviewer`, `/git-accept`, or the main session when a human lets it merge) deletes the remote branch, only after confirming the branch is in `origin/dev`: `git fetch --prune origin && git merge-base --is-ancestor origin/<branch> origin/dev && git push origin --delete <branch>`. The main session returns the checkout to `dev` (`git switch dev && git pull --ff-only`) and deletes the local branch with `git branch -d <branch>` - `-d`, never `-D`, so git refuses if the branch still has unmerged commits. Do not delete the branch of an unmerged PR, a closed PR, or one waiting on `ESCALATE`.

## 6b. Release, Stabilization and Hotfix

Use the skill `release-kit:git-release` (configured in `.claude/release-kit.json`; `CONTRIBUTING.md` § Release Process). The Release manager owns each step that needs a human.

- **Cut (Phase 1, on `dev`):** the skill cuts `release/vX.Y.Z` and writes the CHANGELOG entry on that branch, committing nothing to `dev`.
- **Stabilization:** bugs found in QA are fixed on a `fix/*` branch cut from `release/vX.Y.Z` (`implementer`), PR into that release branch, rebased onto `origin/release/vX.Y.Z` before review. `pr-reviewer` reviews like a PR into `dev` but **does not merge**: the PASS is recorded and the Release manager lets it merge.
- **Release (Phase 2):** the skill runs the mandatory gate, asks the Release manager to confirm, then opens and merges the PR `release/vX.Y.Z -> main` itself and tags. That is the only gate on a PR into `main`.
- **Cleanup of `release/*` and `hotfix/*` branches after merging into `main`:** do not use `gh pr merge --delete-branch`; the merger deletes the remote branch only after confirming it is in `origin/main`: `git fetch --prune origin && git merge-base --is-ancestor origin/<branch> origin/main && git push origin --delete <branch>`, then deletes the local branch with `git branch -d <branch>` - `-d`, never `-D`.
- **Hotfix:** `implementer` fixes on `hotfix/<name>` cut from `main`; the main session runs `code-reviewer` on the diff `origin/main...hotfix/<name>`; then `git-release` handles the CHANGELOG, the mandatory gate, the PR into `main` (Release manager confirms) and the tag.
- **Reconcile `main` into `dev` after each release or hotfix:** branch protection forbids direct pushes to `dev`, so `git-release` opens a `chore/reconcile-main-<version>` PR that merges `main` into `dev` (`devWrites: "pr"`). The Release manager merges it with `--merge` once CI is green; `pr-reviewer` never merges it. It is an exception to the rebase rule: the PR is itself a merge of `main`, and must not be squashed or rebased.
- **Quick release (`quick`, `dev -> main`, no stabilization stage):** the CHANGELOG goes into `dev` through a PR (`devWrites: "pr"`), then the skill opens the PR into `main`. Use only when the Release manager explicitly asks for `quick` and no separate QA is needed.
- **Related config** in `.claude/release-kit.json`: `devWrites: "pr"`, `devMergeCommand: "/git-accept"`, `pushAfterRebase: "force-with-lease"` (`release-kit:git-sync` and `git-mr` push after rebase themselves, overwriting only the branch's exact old head), `prExtraSections` (the PR description sections this process requires; `release-kit:git-mr` pre-creates them).

## 7. Traceability

Requirement codes run through the whole chain: persona (`P-nn`) -> user story (`PRDxx-US-nn`) -> PRD (`PRDxx-FR-nn`) -> design board title -> tech doc section -> plan task -> test -> the `PRD:` line of the feature-map -> PR description. A code with no downstream, or a component with no upstream code, is a finding in any review.

## 8. Recurring Errors and End-of-Module Retro

When the same review comment appears twice, do not add another text rule: turn it into a lint, an architecture test or a generator (`AGENTS.md`).

**End-of-module retro.** The whole team attends. When a module's last PR has merged into `dev`, and before marking *Merged* in `progress.md`, the main session runs the retro:
1. Classify the module's `fix` commits by root cause (a Sonnet agent reads `git log` and `git show` of the module branches).
2. Record each cause group in `docs/agent-error-log.md`, and handle the items in that file's *Pending retro* section. Each member may have met different agent errors; record them all and merge them at the retro.
3. A group seen two or more times becomes an automated check (an S-size task for `implementer`), or the reason it cannot be done is recorded in that section. An automated item is marked "Automated as: ...".
4. Record the module's metrics in `progress.md` § Metrics, with the table from `bun scripts/risk-tier.ts --fix-ratio <module commit range>` and the total agent tokens summed from the `## Verification` sections of the phase PRs.
5. Bundle the module's process changes into one process PR, with the list of personas no story serves any more (§1b), if any.
6. If the tier-1 `fix` ratio is higher than tier-2's, add the paths of the tier-1 `fix` commits to the tier-2 list of `scripts/risk-tier.ts`, in the process PR of step 5.

**Process freeze mid-module.** Process changes (this file, `AGENTS.md`, `.claude/agents/`, `.claude/rules/`) apply only at the retro. They need Tech lead approval. Mid-module, fix only process bugs that block work, with the reason in an entry of `docs/process/decisions.md`. Improvement ideas that arise mid-module go in the *Pending retro* section of `docs/agent-error-log.md`.

**`sa-calibration.md`.** `.claude/sa-calibration.md` records who overrode `solution-architect`. Two members with conflicting taste is a question for the PO, not two contradicting lessons.

**One place per rule.** Each rule has one owning file; elsewhere only point to it as `file § heading`, do not copy the content. `AGENTS.md` is the index and keeps only the rules every session needs right away. If a broken `§` reference appears twice, add a check script to `scripts/test-gate.sh`.

## 9. Process Decisions

The process decision log is `docs/process/decisions.md`, newest on top.

## 10. Module Progress

The progress table is `docs/process/progress.md`: one row per module, one column per stage. The main session is the only one that updates it: in the docs PR when the PRD or tech doc is approved, in the feature PR for the *Plan* and *Code* columns, and it marks *Merged* in the module's next docs PR, only after the end-of-module retro (§8) is done. The *Metrics* table in the same file records six numbers per module: calendar days (from the first PRD draft to the merge of the last PR), PR count, the `fix` commit ratio (`git log`), the `fix` ratio by tier (`bun scripts/risk-tier.ts --fix-ratio`), questions asked to humans, and agent tokens.

**Agent tokens.** Each agent's completion report carries that agent's token count. At the end of each phase, the main session records the total by agent type and tier in the `## Verification` section of the phase PR, for example `implementer t1: 210k; pr-reviewer t1: 45k`. The main session's own tokens cannot be measured, so they are not recorded.
