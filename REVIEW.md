# Review priorities

Review the actual diff, its intended outcome, callers and tests. Follow [the development workflow](docs/development-workflow.md).

Failure means a request reaches the wrong tenant/provider, credentials escape their boundary, usage is charged incorrectly or twice, a stream is corrupted, a VM gains unintended host access, or recovery loses user state.

- Inspect authentication/authorization, tenant and wallet boundaries, idempotency, quota and request-log redaction in `src/`, `worker/`, and `api-kernel/`.
- Trace streaming cancellation, retries and billing together; a successful HTTP response does not prove correct settlement or resource cleanup.
- Treat `deploy/`, `docker/`, `scripts/`, `bin/`, `share/`, Dockerfiles and `.github/workflows/` as sensitive execution or supply-chain paths. Review permissions and whether untrusted PR content can execute with secrets.
- For web changes, inspect loading/error/permission states and screenshots. A build is not interaction evidence.
- Do not mechanically review generated bundles, lockfiles or binaries line by line; review their provenance and relevant source changes. Never skip dependency, migration or executable-artifact risk solely because a file is generated.
- Report only actionable defects, with verified file and line, trigger, impact and a minimal fix direction. Separate introduced defects, pre-existing issues and evidence gaps. Avoid style-only churn.
- Begin with the count of actionable findings, including **0 findings** when clean. Record reviewed base/head, checks inspected and coverage limits. Validate suspicions before reporting them.
- Resolve findings by a verified fix or a written reason. Re-review changed code and affected interactions, not repeatedly discovered unrelated old code. After three rounds, reassess the approach; the limit never waives a defect.
