# Product Decisions

## Confirmed decisions

### Replay architecture
**Decision:** Replay is an overlay on top of the live picture.

Requirements:
- Immediate access to a recent sequence.
- Replay runs independently of the live clock.
- Live streaming and continuous recording continue unaffected.
- Returning to Live must be straightforward and must not disturb the match feed.

### Continuous recording
**Decision:** Record continuously to a storage location selected by the user when starting a tournament.

Reason:
A referee must be able to retrieve the full sequence around a moment later, even when no bookmark or clip was created during the match.

Retention duration and disk-full policy remain to be finalized. A 30-day retention period and deletion of the oldest recording when storage is full have been discussed, but are not a locked decision.

### Bookmarks
- Keep bookmark controls visible but compact.
- Prefer a small bottom drop-up style control/drawer.
- Creating a bookmark must not automatically open the bookmark drawer.

### Live status
- The Live button/indicator should be green when live is active.
- Help text should identify the computer address as “PC IP”.

### Installation safety
- Keep production and test installations separate.
- Production must not be modified as part of Rev.8 testing or repair.


## Rev.9 requirements direction — user-confirmed priority (2026-11-08)

**Requirement:** Rev.9 must be very easy to install on both Windows and Linux.

This is a product requirement, not yet an approved technical implementation choice. The requirements/design phase must compare suitable packaging and service-management approaches and explain the installation/update/uninstall experience for both operating systems.

Acceptance criteria must cover:
- A first-time user can install and launch the application without manually editing absolute paths or YAML/Caddy configuration.
- The application can be installed in a user-selected or otherwise clearly documented location.
- Recording storage can be selected independently of the application installation directory.
- Required runtime components and versions are checked or provisioned in a predictable, supportable way.
- Start, stop, update, diagnostics, and uninstall procedures are documented for Windows and Linux.
- No installer or update process can overwrite a separate existing production/test installation or delete user recordings without explicit confirmation.

The following remain open design decisions, not locked choices: native installers versus a portable package/container approach; service supervision model; update channel; privilege requirements; supported Linux distributions; retention duration and disk-full policy.
