# Roadmap — Draft v0.1

**Current status:** Planning only. Reserve Codex for implementation after higher-priority projects.

## M1 — Foundation
Self-contained WPF app; load/validate local YAML; show resolved review screen with Install/Skip; mock execution and tests.
**Done when:** A sample profile loads, invalid input is explained, and selections do not change YAML.

## M2 — Winget execution
Package detection, install/skip, progress, logs, failures, and existing-install detection.
**Done when:** On a Windows test VM, selected packages install or are correctly reported already installed; skipped ones are untouched.

## M3 — Guided workflows
PowerShell steps, manual instructions, automatic/manual continuation, dependencies, and sequential unattended mode.
**Done when:** A multi-step example respects checkpoints, dependencies, retries, and unrelated work.

## M4 — Persistence and recovery
Durable journal, interruption handling, restart countdown, resume-after-sign-in, loop protection.
**Done when:** Test runs recover from simulated crashes and real VM reboots without unsafe duplicate execution.

## M5 — Windows Update
Native Windows Update API integration, update category policy, restart/rescan loops.
**Done when:** VM update runs report outcomes and resume safely; bounded retries prevent loops.

## M6 — Profiles and recipient setup
Reusable profile components, local/HTTPS configuration, prepared Simple Mode entrypoints, trusted-command approval.
**Done when:** A recipient can start an approved setup without editing YAML or signing into the author's accounts.

## M7 — Inventory and migration
Installed-software scan, full inventory export, migration plan, selective Add to Config, shareable reports.
**Done when:** Discovered software can be added deliberately without automatically modifying permanent profiles.

## Release considerations
Unsigned portable builds, clear SmartScreen/UAC guidance, release notifications, Windows 10/11 smoke tests, and operator documentation.

Each milestone should be a separate reviewable implementation slice. No code or release work begins solely because these documents are committed.

## Central management extension (confirmed 2026-11-04)
The seven Windows milestones remain the foundation. Add independently scoped server/integration slices:
- **S1 Django admin/API foundation:** profiles, YAML drafts, validation, published revisions, hashed access-code verifiers, optional expiry/usage limits and revoke/reactivate/rotate.
- **S2 Recipient retrieval:** single-download website entry point, password entry in Windows app, HTTPS retrieval of latest published signed package, immutable snapshot and offline local YAML support.
- **S3 Sessions and notifications:** authenticated installation sessions, idempotent completion reporting, default deactivate-on-success policy, locally queued offline reports, admin history, optional n8n webhooks.
- **S4 Admin polish:** improve beyond Django Admin only where actual usage shows friction.

Security acceptance: VPS never initiates client execution; untrusted/modified script packages are rejected; server cannot silently mutate an active snapshot. Build S1/S2/S3 after standalone workflow foundation is testable.
