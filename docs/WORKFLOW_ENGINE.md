# Workflow Engine Contract — Draft v0.2

This document specifies behavior, not a particular implementation. No code has been written.

## Concepts
- **Profile**: reusable selection of task definitions.
- **Task**: user-visible unit with stable ID, default Install/Skip choice, and ordered steps.
- **Step**: atomic operation executed by one adapter.
- **Run plan**: immutable snapshot of resolved profile, selections, dependencies, and approvals.
- **Run journal**: durable history of state transitions and operation outcomes.

## Task/step states
Pending -> Ready -> Running -> Succeeded
Pending/Ready -> Skipped (user selection)
Running -> Failed or Interrupted
Running -> AwaitingUser (completed step with manual continuation)
AwaitingUser -> Succeeded (explicit user confirmation)
Failed -> Ready (explicit retry)
A task is Succeeded only when every required selected step succeeds.
A skipped prerequisite blocks dependents; it does not count as success.

## Execution algorithm
1. Load, validate, and resolve profile includes; reject cycles, duplicate IDs, missing dependencies.
2. Present plan with default Install/Skip selections. Recompute dependency warnings when selections change.
3. Freeze plan, show exact privileged actions and their sources, and request approval before executing.
4. Execute ready steps sequentially. Never advance within a task before its prior step completes.
5. In guided mode, pause on manual checkpoints. In unattended mode, defer blocked tasks and continue independent ready tasks.
6. On error, record full outcome; do not automatically retry non-idempotent actions. Independent work may continue.
7. Persist transition before any irreversible action or restart request.
8. On resume, inspect interrupted steps and reconcile observable results; otherwise ask user before retrying.

## Continuation semantics
- `automatic`: proceed after successful completion.
- `wait-for-user`: action runs, then a confirmation button is required before the following step.
- Manual instruction step: display text/links and wait for completion or skip decision.
- `stop`: finish task after current step.
- An unattended session never bypasses a wait-for-user checkpoint to run a dependent step.

## Failure policy
- Failed step: stop its task; mark dependent tasks blocked; continue unrelated tasks.
- User can Retry, Skip Task, or End Run; show effect on dependents.
- Default automatic retry count is zero for scripts and installations until retry-safety is explicitly defined.
- Preserve stdout/stderr, exit codes, and adapter diagnostics.
- A cancelled task does not silently become successful.

## Restart policy
- Save run journal and resume registration before triggering a restart.
- Show 30-second countdown with Restart Now / Don't Restart Now.
- When postponed, keep restart-dependent tasks blocked; other safe tasks may proceed.
- Resume after interactive sign-in, never before sign-in in MVP.
- Track restart attempts and pause if a bounded loop is detected.
- Clean up resume registration on terminal run completion.
- Restart must not forcibly close unrelated applications.

## Windows Update
- Native Windows Update API adapter, not arbitrary scripts as default.
- Configurable standard updates; optional drivers/previews are separate explicit choices.
- Rescan after reboot; bound scan/restart cycles.
- Respect managed Windows Update policies; report unsupported/blocked conditions.

## Idempotency and recovery
- Winget steps: inspect installed package state before installation; check again after interruption.
- PowerShell: default `retry_policy: manual`; do not blindly rerun after crash.
- Manual steps: persist awaiting-user status.
- Windows Update: reconcile through update status/history.
- Every state mutation is journaled atomically; UI reflects journal rather than inferring success.

## Test cases required before real OS integration
1. Skipping an optional package does not modify YAML.
2. Skipping a prerequisite blocks dependents and explains why.
3. A manual checkpoint blocks its next step, but independent tasks continue unattended.
4. A failed installation leaves unrelated tasks runnable.
5. Crash during PowerShell does not silently rerun script.
6. Restart saves progress; resume does not repeat completed steps.
7. Repeated restart requirements reach a safe pause rather than loop forever.
8. An untrusted remote script cannot execute without explicit review/approval.

## Explicit non-goals for MVP
Concurrent installers, pre-login background service, full offline caching, credential storage, automatic driver recovery, and unattended bypass of UAC/SmartScreen.

## Remote profile acquisition and reporting contract (2026-11-04)
1. App launches; recipient enters setup access code.
2. Server resolves code to profile and returns the latest published version, not a previously downloaded version.
3. App verifies signed package (including scripts), checks schema compatibility, and stores immutable snapshot before execution.
4. Recipient reviews/confirms; execution is local. A running session never receives remotely initiated actions or live config mutations.
5. Resume after restart or interruption uses saved snapshot.
6. On successful terminal completion, persist outcome locally and submit authenticated, idempotent completion report to server. Server then deactivates code when its policy is `after_success`.
7. Failure/interruption leaves code active. If reporting is offline, retry later; the server shows pending/last-seen status rather than inventing success.
8. Revocation prevents future configuration retrieval; it does not retroactively cancel an already snapshotted run.

Additional acceptance tests: new publication after executable download is retrieved at code entry; publication during a run does not change snapshot; duplicate success events deactivate once; forged success is rejected; failed run does not deactivate; offline success retries reporting; revoked code cannot fetch a new config; n8n failure does not affect local completion.
