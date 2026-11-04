# Architecture — Draft v0.1

## Technology
C#/.NET desktop application with WPF; self-contained Windows distribution. Separate testable workflow/core libraries from GUI and OS adapters.

## Components
1. **WPF UI:** Simple/Advanced modes; profile loading, review selections, status, manual checkpoints, inventory, reports.
2. **Configuration:** YAML parser, schema validation, includes/profile composition, versioning, immutable per-run resolved plan.
3. **Workflow engine:** Sequential step state machine, dependencies, automatic/manual continuation, unattended scheduling, retry/skip semantics.
4. **Executors:** winget, PowerShell, Windows Update Agent API, manual tasks, restart, file/URL actions.
5. **Persistence:** durable run journal, logs, completion reports, and inventory; atomic writes and crash recovery.
6. **Elevation boundary:** UI runs unelevated; approved tasks run in elevated worker with constrained, auditable request protocol.
7. **Distribution:** public release; optional local or HTTPS config; prepared Simple Mode setup entrypoint.

## Execution invariants
- Never mark a step successful before verifying completion.
- Never execute a dependent step when its prerequisite failed, was skipped, or awaits confirmation.
- Unattended mode may continue independent tasks while deferring manual checkpoints.
- After a crash or restart, re-evaluate in-flight operations before retrying; do not blindly rerun non-idempotent scripts.
- Persist a checkpoint before requesting restart; resume after interactive sign-in and remove resume hook after completion.
- Never silently execute newly downloaded PowerShell solely because a URL is password-protected.
- Detect winget availability and provide a clear prerequisite path.

## Test strategy
Unit tests for schema, dependency graph, transitions, selection overrides, and recovery. Mock adapters for winget, PowerShell, Windows Update, elevation, and restart. Integration/smoke tests on real Windows 10/11 VMs, especially UAC, winget bootstrap, interruption, and reboot loops.

## Deferred design
Precise privilege protocol, storage format, configuration integrity/signing strategy, and Windows Update selection semantics require implementation design before coding.
