# Development and review workflow

This repository uses layered verification adapted from [How the Claude Code team does code review](https://claude.dev/blog/solving-code-review-at-anthropic/) (2026-10-15). The objective is fewer real defects and less human review effort without increasing reverts or incidents. [AGENTS.md](../AGENTS.md) is the concise entry point; [REVIEW.md](../REVIEW.md) defines failure modes.

## Before a PR

1. State the problem, acceptance evidence, affected contracts, risk, human owner and recovery plan. Prefer small independently reversible changes.
2. Implement and run the relevant checks below. Add meaningful regression coverage for bug fixes. For UI behavior, include screenshots and interaction evidence. Record command output and limitations; skipped checks are not passes.
3. Ask an independent AI agent with fresh context to review the diff, intended outcome and surrounding code. Give it the applicable instructions and base/head, not just the author's explanation. Explicitly review security boundaries. Validate suspected findings, fix real defects and retain dispositions.
4. Complete the PR template. An AI reviewer can run locally; evidence must name the tool/model, revision, scope, findings and unresolved gaps. An unattended GitHub AI reviewer is not configured by these files.

## Verification catalog

Use Node 22, Go 1.25.x, Python 3 and pnpm 10 as specified by the existing CI. Match checks to changed areas and keep the full CI on the PR. Run only in an isolated checkout without production credentials.

| Area | Commands from the repository root |
| --- | --- |
| Documentation only | Review assertions and relative links; `git diff --check` |
| Node gateway | `npm ci`; `npx biome format`; `npm run test:unit` |
| Worker and API kernel | `(cd worker && go test -race ./... && go vet ./...)`; `(cd api-kernel && go test -race ./... && go vet ./...)`; review `gofmt -l worker api-kernel` output |
| Web | `pnpm -C web install --frozen-lockfile`; `pnpm -C web format:check`; `pnpm -C web test`; `pnpm -C web build` |
| ARM64 host preparation | `python3 -m unittest discover -s test/deploy -v`; native container/binfmt checks run only on a disposable ARM64 runner |
| E2E and release artifacts | Follow the test fixtures and existing CI; do not substitute production services or publish a release for a test |

The existing `ci` workflow covers Node, Go, web, prebuilt ELFs and native ARM64 deployment. Keep its job names and checks. It listens for `merge_group` so a supported merge queue can run the same checks. A fork may require enabling Actions before any workflow runs. Missing workflows/checks are a readiness gap, not a green result.

## Review and merge

Risk follows the change. Documentation/copy without behavior changes is generally low; ordinary behavior changes are medium. Auth, permissions, billing, secrets, migrations, host/VM execution, CI, releases and architectural contracts are high risk regardless of size. A documentation change that alters a security policy is also sensitive.

Keep a human owner on every change. Initially retain human sign-off even for low-risk changes; only relax it for trusted authors after the owner approves the policy and rollback/incident evidence supports it. Medium changes need one responsible human to review the final revision and evidence; this can be the owner. High-risk changes need the owner and a second competent independent human. AI approval never fills a human slot. The PR author cannot approve their own PR in GitHub; owner sign-off outside GitHub is not a substitute for a required independent GitHub approval. Preserve stricter configured rules and add an eligible collaborator before requiring their server-side approval. If a required reviewer is unavailable, leave the merge pending rather than faking approval.

The initial trusted-author list is `xujinyang`. Other authors need an additional qualified human review even when their risk tier would allow owner-only review.

Every AI/security finding must be fixed or dismissed with a reason, and all review threads resolved before merge. Record reviewed revisions and revalidate affected code after new commits or conflict resolutions. Normally use one full review and focused verification of fixes. After early passes, review only new diffs, affected interactions and unresolved findings. At three rounds, reassess the design or involve the owner; do not loop or auto-waive findings.

Enable required CI checks, resolved-discussion rules, stale-approval dismissal and sensitive ownership reviews using supported repository settings. Verify checks exist and run before making them required. These settings are not applied by a PR template or CODEOWNERS file. At onboarding, the default branch had no protection/rulesets; unattended AI and security-review services remain an explicit integration step. Do not make a missing service silently succeed, grant a PR secret access, or execute PR code in a privileged `pull_request_target` job.

Use merge queue when supported. Otherwise update against current `main` and re-run applicable checks before merging. A merge-group trigger does not itself enable a queue. Do not weaken protection to make a merge convenient.

## Release and recovery

Use flags for reversible behavior where appropriate; canary/internal validation precedes wider rollout. Name the owner, signals, abort thresholds and rollback command in the PR or release record. Reverting source is not sufficient for destructive database changes: validate backup/restore or a forward-compatible recovery plan first. Production deployment remains a separate authorized operation.

After an incident or revert, record the failing PR, impact, recovery and which layer should have caught it. Add a regression/evaluation case and improve tests, lint or review instructions. Include known-clean changes when measuring false positives. Re-score changes to reviewer prompts/models/tools against these cases. Track review rounds, human approvals/wait time, false positives, revert and incident rates; automation is successful only when quality guardrails hold.
