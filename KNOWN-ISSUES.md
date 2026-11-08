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


## Addendum — 2026-11-08: Issues and design gaps found in static Rev.8 package review

These are source/configuration observations, not all reproduced runtime failures.

| ID | Issue / gap | Status and Rev.9 implication |
|---|---|---|
| JR-13 | Windows-only installation assumptions | CONFIRMED IN PACKAGE. Absolute Windows paths and PowerShell-only scripts; no Linux setup. Rev.9 must support easy Windows and Linux installation. |
| JR-14 | Recording health is not demonstrated by UI | STATIC OBSERVATION. “Configured” is not equivalent to actively recording. Show verified backend/file health, errors and disk state. |
| JR-15 | Bookmark persistence/association is browser-local | STATIC OBSERVATION. localStorage and fixed camera: 0 metadata do not provide a robust shared tournament event model. |
| JR-16 | Bookmark creation expands drawer | SOURCE CONFIRMED. Conflicts with the product decision that creating a bookmark must not open the drawer. |
| JR-17 | Seek completion is not explicitly awaited | STATIC OBSERVATION. loadSpan sets currentTime after canplay but does not explicitly await seeked/presented frame before continuing. Validate with deterministic playback tests. |
| JR-18 | Frame-step can be approximate | SOURCE CONFIRMED. Fallback uses 1/30 second when a neighboring recorded frame timestamp is unavailable; exact frame-step is not guaranteed. |
| JR-19 | Network service exposure needs security design | STATIC OBSERVATION. Rev.8 test MediaMTX listeners bind wildcard interfaces, including API/metrics. Define least-privilege binding, access control and firewall requirements. |
| JR-20 | Stop script path-boundary validation | STATIC OBSERVATION. Raw StartsWith(testRoot) should be replaced by normalized path-with-directory-boundary validation before treating it as a strong safety guard. |

The Rev.8 archive also contained `auto.key`. Keep private keys out of future source bundles, public repositories and releases. Runtime logs and server executables were not included, so their behavior remains outside this static review.
