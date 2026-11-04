# Codex Handoff — Planning Only

## Goal
Implement a portable Windows 10/11 setup manager using C#/.NET WPF. Winget is the default installer; tasks can also contain PowerShell, manual checkpoints, Windows Update, and managed restarts.

## Read first
1. docs/PRODUCT_REQUIREMENTS.md
2. docs/ARCHITECTURE.md
3. docs/WORKFLOW_ENGINE.md
4. docs/CONFIGURATION_CONTRACT.md
5. docs/ROADMAP.md

## Working rules for implementation
- Work on one GitHub issue/milestone slice at a time; do not silently expand scope.
- Begin with the configuration validator, pure workflow engine, and tests; OS adapters can be mocked.
- No real installation, elevation, reboot, or Windows Update actions in tests unless explicitly marked Windows integration tests.
- Keep GUI separate from workflow logic and elevated worker.
- Include test results and exact manual Windows smoke-test instructions in each PR.
- Do not merge PRs without user review.
- Document deviations from the proposed configuration contract before implementation.
- Never assume successful execution from process exit alone when adapter verification is possible.

## Suggested first coding slice
M1a: solution skeleton, YAML schema validation, sample config, unit tests for duplicate IDs, invalid executor, include cycle, missing dependency, and Install/Skip session overrides. No GUI or actual winget calls yet.

## Cost discipline
Provide bounded acceptance criteria and exact files to change in each issue. Avoid asking Codex to design and implement the entire product in one task. Reserve live Windows integration for dedicated milestones.
