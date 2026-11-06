# Known Issues — Rev.8

Status snapshot: 2026-11-06. These are reported or previously observed issues, not all independently reproduced in the latest test session.

| ID | Issue | Notes |
|---|---|---|
| JR-01 | Live delay is too high / not adjustable as expected | Approx. 10 seconds was reported. |
| JR-02 | Instant Replay cannot find completed recordings | UI reports “No completed recording found yet”. Check archive discovery and segment finalization. |
| JR-03 | HLS segment retrieval can return HTTP 401 | Caddy-mediated segment requests previously returned “session not found”, although playlists returned 200. |
| JR-04 | Bookmark click opens the bookmark drawer | Bookmark creation should only create the marker; it should not automatically open the drawer. |
| JR-05 | Recording filename may become `.mp4.mp4` | Verify MediaMTX filename/extension behavior before changing the recording path. |
| JR-06 | Pause snaps the playhead to live | A paused replay must remain at the selected timestamp. |
| JR-07 | Frame stepping jumps to live | Frame-by-frame inspection must keep the replay position stable. |
| JR-08 | Actual delay differs from configured delay | Measure and identify whether the discrepancy is in ingest, HLS, player, or UI logic. |
| JR-09 | Zoom does not follow the selected area | Zoom should use the chosen focal point/area, not only the center. |
| JR-10 | Live status indicator flickers | The active Live indicator should remain green while the live source is healthy. |
| JR-11 | False “No signal” overlay | Do not show a no-signal state while a valid live picture is being rendered. |
| JR-12 | Help text is ambiguous | Refer to the “PC IP” rather than simply “IP address”. |

## Investigation rules

- Inspect the exact current Rev.8 source files before attributing a cause.
- Separate confirmed reproduction from static-code observations and hypotheses.
- Consolidate related issues before proposing a repair.
- Do not change the production installation.
