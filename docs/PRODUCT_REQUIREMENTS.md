# Product Requirements — Draft v0.1

## Purpose
Make Windows 10/11 computer setup repeatable for personal PCs, spouse's PCs, two school environments, and a boss's PC; support remotely shared, simple setup experiences.

## Core requirements
- Portable downloadable Windows GUI, optionally run from USB; no personal GitHub/Bitwarden authentication required to launch.
- Public generic app, separate YAML configuration; local profiles with optional password-protected HTTPS retrieval from user's site. Do not put secrets in configurations.
- Profiles composed of reusable tasks. Each task may contain ordered steps: winget install, PowerShell, manual instructions, download/open URL, Windows Update, and restart.
- Each step supports automatic continuation or an explicit manual checkpoint. Respect prerequisites and do not skip dependencies when deferring interactive steps.
- Review screen: Install/Skip per application and category, default selections from YAML; overrides affect only the current session unless explicitly saved.
- Simple Mode for prepared recipient setups; Advanced Mode for profile choice, logs, inventory, and editing.
- Sequential unattended execution of eligible tasks, deferring blocked/manual work; record failures, allow retry, and continue unrelated tasks.
- Managed restart with 30-second prompt, Restart Now / Don't Restart Now, saved progress, and resume after sign-in. Avoid restart loops.
- Windows Update integration, including repeated scan after restart, with configurable update categories.
- Installed software inventory before reformat, including non-winget software when detectable; retain full inventory and select items for reinstall. Add to Config is explicit, never automatic.
- Local history, exportable/shareable reports, recovery after interruption.
- Internet-first: no full offline installer cache or network-driver recovery in MVP.
- No credential or license storage. Account sign-in and activation are manual; personal profile installs Bitwarden early.
- Application update notification; no silent self-update during active setup.
- No paid code-signing dependency; expect possible SmartScreen warnings.

## Security and usability
- Explain Windows UAC/SmartScreen clearly; never bypass OS security.
- Validate YAML, identify command origins, show privileged actions before approval; password-protected URL is not sufficient command integrity protection.
- Prefer least-privilege UI with one elevated worker session for approved tasks.
- Do not force-close user applications during restart.

## Open questions / future design
- Exact task schema and approval model.
- Exact implementation of shareable packages and URL authentication.
- Windows 10 compatibility baseline and supported winget bootstrap scenarios.
- Definition of 'required' tasks in Simple Mode.
- Network recovery and full offline media are out of MVP scope.

## Confirmed centralized distribution and administration decisions (2026-11-04)
- Two independently usable components: local Windows WPF installer and small Django administration/API service on the VPS. Local YAML remains supported.
- Recipient uses one website entry point (e.g. `millsdaniel.com/computersetup`) to download the installer, then enters a setup access password in the app. No second download or separate configuration handling.
- Password entry fetches the latest **published** configuration at run time, even when the executable was downloaded days earlier. Draft edits do not affect recipients.
- A setup access password maps to a profile, can be user-chosen or randomly generated, and is managed via authenticated admin UI. Multiple credentials can map to one profile.
- New access codes default to **deactivate after verified successful installation**. Other policies: indefinite until manual revocation; deactivate on first retrieval. Optional expiration and usage limits; reactivation/rotation supported.
- Failed or interrupted installations do not deactivate access codes. Successful installation is recorded locally even without connectivity; server deactivates only after authenticated completion event arrives.
- The recipient sees minimal technical warnings in Simple Mode; integrity/compatibility checks are automatic. OS UAC remains intact.
- VPS may manage YAML drafts, validation, versioning, publishing, credentials, history, and n8n event notifications. It cannot initiate installs, execute client commands, or alter active runs.
- The server stores access-code verifiers securely, not plaintext codes. Avoid transmitting codes in event reports.
- A completed run's reporting is best effort with durable local retry; server outage must not prevent a local setup from finishing.
