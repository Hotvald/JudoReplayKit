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
