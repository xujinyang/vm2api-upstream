# Development instructions

Read [REVIEW.md](REVIEW.md) and the [development workflow](docs/development-workflow.md) before changing this repository.

- Keep changes small and preserve the gateway's request, streaming, tenant-isolation and settlement contracts. Diagnose the root cause before changing compatibility behavior.
- Define acceptance checks, risk and recovery before implementation. Run applicable checks from the workflow and report actual output, skips and failures.
- Obtain fresh-context independent AI review and explicit security review before opening a PR. Record the reviewed revision, findings and dispositions. Three review rounds require reassessing the approach; unresolved defects still block merge.
- Authentication, tenant boundaries, billing, credentials, database migration, VM execution, CI and releases require human review of the final revision. Bot approval is not human approval.
- Every PR identifies its human owner, Testing, Risk, AI/security review and How to revert. Do not claim a policy file enables GitHub merge protection or an unattended reviewer.
- Use isolated fixtures and disposable test services. Never use production credentials, wallets, databases, upstream subscriptions or deployment targets for CI. A source PR does not authorize production operations.
- Preserve upstream provenance and existing release workflows. Do not run release or guest-image publication as part of verification.
