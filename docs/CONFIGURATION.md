# Configuration — Draft v0.1

YAML is the human-editable source of truth. The UI presents a resolved plan and supports one-session Install/Skip overrides. An editor may be added later.

Illustrative syntax **only**, not a finalized parser contract:

```yaml
schema_version: 1
profile:
  id: personal
  name: Personal Computer
includes:
  - common-windows
tasks:
  - id: bitwarden
    label: Install Bitwarden
    default: install
    steps:
      - type: winget
        package_id: Bitwarden.Bitwarden
        continue: automatic
  - id: windows-settings
    label: Configure Windows
    default: install
    steps:
      - type: powershell
        script: scripts/windows-settings.ps1
        continue: wait-for-user
  - id: updates
    label: Windows Update
    steps:
      - type: windows-update
        restart: allowed
```

## Design rules
- Stable IDs for profiles/tasks/steps; explicit schema version.
- Reusable task definitions and profile includes, with cycle/duplicate detection.
- Dependencies and step continuation are separate concepts.
- UI selection changes do not mutate YAML unless user explicitly saves.
- Required/default/optional selection policy must be explicit.
- Inventory and migration plan are separate from permanent profile.
- Scripts must be locally packaged or integrity-verified and explicitly approved; never include passwords or license keys.
- A resolved run stores the exact config version and selected steps for reproducibility.

## To finalize
Field names, schema validation, task-level versus step-level dependencies, PowerShell arguments, failure policies, Windows Update categories, and remote authentication format.
