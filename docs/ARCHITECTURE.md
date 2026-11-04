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

## Confirmed deployment boundary (2026-11-04)
- **Windows app:** WPF/.NET, executes all installation actions locally only after the local user initiates setup. Works with local YAML without VPS.
- **VPS:** Django admin + API manages YAML drafts/published revisions, access credentials, installation session records, and optional n8n notification delivery. No remote execution, control agent, inbound client access, or mid-run command injection.
- **Single recipient entry point:** website hosts/links the installer; the installed app asks for the access code and retrieves latest published profile over HTTPS. Website may proxy/redirect the executable while retaining a one-download experience.
- **Trust boundary:** verify signed configuration packages and all referenced scripts. Signing private key remains off the VPS; verification public key ships in trusted application. VPS compromise alone must not allow arbitrary replacement of executable commands in verified packages. An app binary replaced at the download source remains a separate distribution risk.
- **Snapshot lifecycle:** authenticate -> retrieve latest published config and associated assets -> verify integrity/compatibility -> persist immutable local snapshot -> review/approve -> execute from snapshot -> persist/report outcome. Reboots and restarts resume the same snapshot; never silently fetch changed config into an active run.
- **Credential lifecycle:** access code points to profile, not directly to immutable YAML. Server authenticates without storing plaintext, creates scoped installation session and accepts authenticated status events. Completion event is idempotent; verified success transitions default access code to inactive/completed. No completion event means no automatic deactivation.
- **n8n:** optional downstream webhook from authenticated server, not a public webhook embedded in client. Queue/retry events; notifications never block installation.
- **Admin:** use Django Admin first; custom UI only when needed. Draft/published revision separation and audit log.
