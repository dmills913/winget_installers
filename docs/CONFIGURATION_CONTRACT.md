# YAML Configuration Contract — Draft v0.2

This is a **proposed authoring contract**, not yet a committed parser API. Finalize field names during M1 before coding executors.

## Example profile
```yaml
schema_version: 1
profile:
  id: personal-pc
  name: Personal PC
includes:
  - common-windows
tasks:
  - id: bitwarden
    label: Bitwarden
    category: Essentials
    selection: default
    steps:
      - id: install
        type: winget
        package_id: Bitwarden.Bitwarden
        continuation: automatic

  - id: workstation-settings
    label: Windows preferences
    category: Configuration
    selection: optional
    steps:
      - id: apply
        type: powershell
        script: scripts/preferences.ps1
        continuation: wait-for-user
      - id: verify
        type: manual
        instructions: Verify the requested settings.
        continuation: wait-for-user

  - id: updates
    label: Windows Update
    category: System
    selection: default
    steps:
      - id: run
        type: windows-update
        policy: standard
        allow_restart: true
        continuation: automatic
```

## Proposed fields
- `schema_version`: required integer, reject unsupported major versions.
- `profile.id/name`: stable machine-readable ID and display label.
- `includes`: reusable local component IDs; detect cycles and duplicate task IDs.
- `tasks[].id/label/category`: unique task ID, display name, grouping.
- `selection`: `required`, `default`, or `optional`; maps to checked and skippable UI behavior.
- `depends_on`: optional list of task IDs; dependencies must succeed before execution.
- `steps[].id/type/continuation`: unique step ID within task; executor type and next-step behavior.
- `winget.package_id`: explicit stable package ID, not a fuzzy search string.
- `powershell.script`: relative path inside trusted configuration package; no arbitrary remote script URL.
- `manual.instructions`: text shown in GUI.
- `windows-update.policy`: `standard` initially; optional categories deferred until tested.

## Constraints
- Relative file references cannot escape the package root; reject path traversal.
- Reject unknown executor types and malformed dependencies.
- Remote configurations must be treated as untrusted until reviewed/approved.
- A password-protected HTTPS endpoint provides access control, not command authenticity.
- No passwords, tokens, or license keys in YAML.
- Freeze resolved configuration plus content hashes in each run record.
- UI Install/Skip overrides do not alter source files unless user explicitly saves.
- Keep installed-software inventory separate from desired-state profile.

## Open schema design questions for M1
- Define precise component include resolution and precedence.
- Define conditional steps, if needed, only after concrete use cases.
- Define script parameters and elevation request format.
- Decide whether user edits are saved as YAML patches or fully rewritten files.
- Establish package format for prepared Simple Mode entrypoints.

## Distribution and publishing (confirmed 2026-11-04)
- YAML is the common portable configuration format for local and server-delivered profiles.
- Django maintains drafts and explicit published revisions. Only latest published revision is served to new authenticated sessions.
- Profile access codes are **server metadata**, not embedded in YAML. Multiple codes can point to the same profile.
- The downloaded package includes YAML and referenced scripts/assets; cryptographic integrity covers the entire executable-content set.
- Each run snapshots the exact published revision and assets; active runs do not change after a later publication.
- Inline PowerShell commands and packaged `.ps1` files should both be supported in the final schema. Keep `manual` vs `automatic` execution separate from `optional` selection; exact field names remain to be finalized.
