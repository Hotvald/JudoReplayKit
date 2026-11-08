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


## Rev.9 specification review — open, not approved (2026-11-08)

DeepSeek's 12-part Rev.9 Requirements Specification has been received. It is a proposal, not an approved implementation baseline. None of its proposed technical choices or owner decisions are confirmed merely by appearing in that document.

The following points must be resolved in a corrected specification before approval:

1. **Replay request duration:** FR-022/FR-023 require the selected window (for example, exactly 15 seconds), while Part 5.3 also permits a full-span request and its pseudocode uses the full span's start/duration. Define one contract and make all requirements, pseudocode and tests agree.
2. **Bookmark persistence service:** If `bookmarks.json` is the server-side source of truth, specify the server API/process, atomic writes, concurrency handling, schema validation, and access control.
3. **Network bindings:** Distinguish RTMP camera-ingest listeners from browser UI, API and metrics listeners. A phone publishing over the LAN cannot connect to an RTMP listener bound only to 127.0.0.1.
4. **Recording extension:** Verify the proposed extensionless `recordPath` against pinned MediaMTX v1.21.3 in an isolated test; do not assume it fixes the observed `.mp4.mp4` issue.
5. **Bookmark post-roll:** Define how a bookmark at click time obtains the requested post-event footage, including recording segment finalization and delayed availability.
6. **Camera synchronization:** Define and test how independent camera clocks/encoders achieve the proposed ±500 ms alignment.
7. **Disk-full semantics:** Clarify what “stop cleanly, no partial write” means when the disk becomes full during a segment, and how incomplete files are detected/recovered.
8. **Performance targets:** Keep proposed CPU/RAM/latency numbers labelled provisional until tested on a defined reference machine.

The following remain open owner decisions from the DeepSeek proposal: retention policy; disk-full/deletion behavior; supported Linux distributions; Windows installer technology; network exposure defaults; LAN authentication; clock synchronization requirement; Caddy `header_down Location`; error wording; replay preload policy; and root cause of the 25.001-second discrepancy.

**Confirmed existing product decisions remain unchanged:** Option A replay overlay independent of live/recording; continuous recording to user-selected storage; compact bookmarks that do not auto-open the drawer; strict separation/protection of production and test installations; easy installation on Windows and Linux as a product requirement. No new retention, installer, authentication, or binding implementation choice has been approved.
