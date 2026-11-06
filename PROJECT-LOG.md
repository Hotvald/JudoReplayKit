# JudoReplayKit — Project Log

Last updated: 2026-11-06

## Purpose

JudoReplayKit is a portable Windows application intended to help judo referees review match sequences without interrupting the match, live stream, or continuous recording.

The application UI and user-facing documentation are to be written in English. Development discussions may be in Danish.

## Core decisions

- **Replay model:** replay is an overlay on top of the live picture. Replay must be independent of the live stream and recording.
- **Immediate access:** replay should be available with minimal delay, especially for quick confirmation of a referee decision.
- **Continuous recording:** video must be recorded continuously to a user-selected storage location so a sequence can be retrieved later, even if no bookmark was created at the time.
- **Bookmarks:** bookmarks mark moments for later retrieval. Creating a bookmark must not automatically open the bookmark drawer.
- **Production safety:** the known-working production installation must not be changed during testing or repair work. Development changes are isolated in the Rev.8 test installation.

## Installation separation

- Production: `D:\JudoReplayKit` — protected; do not modify during test work.
- Rev.8 test: `D:\JudoReplayKit_Rev8_Test` — isolated test installation.

## Rev.8 status (2026-11-06)

The isolated Rev.8 test installation was running during the latest recorded checks.

Verified:
- Web application returned HTTP 200 on the test web port.
- MediaMTX API returned HTTP 200.
- Playback list endpoints for cam1 and cam2 returned HTTP 200.
- cam1 was Ready and Available with an RTMP source.
- cam1 live picture was visible in the browser.
- Continuous recording was confirmed by file growth of 4,412,816 bytes over 10 seconds.
- cam2 was offline because its phone was switched off; this was expected.

Not yet resolved:
- Live delay is approximately 10 seconds and does not match the configured value.
- Instant Replay reports that no completed recording was found.
- HLS segment retrieval through Caddy has previously returned HTTP 401 (session not found).
- Pause and frame stepping can snap the playhead back to live.
- Zoom does not reliably follow the selected area.
- Live status indicator can flicker and a false “No signal” overlay has appeared.
- Bookmark creation opens the bookmark drawer automatically.
- Recording filenames may receive a duplicate `.mp4` extension.
- Help text should say “PC IP” rather than the ambiguous “IP address”.

## Working rules for contributors and AI assistants

1. Do not modify the production installation.
2. Do not stop or kill running production or Rev.8 services without a clear, approved reason.
3. Do not repeat tests already marked as verified unless a specific new change makes them necessary.
4. Inspect the exact current Rev.8 files before proposing code changes. Do not assume an old file is current.
5. Collect and consolidate defects before editing. Present a repair plan for approval before applying repairs.
6. Distinguish verified facts from hypotheses.
7. Do not publish private chat exports, credentials, tokens, personal data, or match recordings in this public repository.
