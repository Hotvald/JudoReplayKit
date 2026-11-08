# JudoReplayKit — ChatGPT's independent review of the DeepSeek Rev.9 specification

**Date:** 2026-11-08  
**Author/source:** ChatGPT, based on the complete DeepSeek Rev.9 specification pasted by the user and the JudoReplayKit project history available in this project.  
**Status:** Preliminary independent review. This is not the final three-way comparison; Gemini's review is still pending.  
**Scope:** Requirements and architecture review only. No code, scripts, configuration, services, or installations were changed.

## 1. Overall assessment

The DeepSeek document is a useful and broad requirements proposal. It covers the right major areas: replay behavior, continuous recording, storage, bookmarks, installation, security, diagnostics, acceptance testing, risks, and implementation planning.

However, it is not yet internally consistent or sufficiently specified to serve as an implementation-ready baseline. Several requirements, architecture statements, and sample flows do not describe the same behavior. Some operational claims also need tests against the exact pinned runtime rather than assumptions.

The user's established core requirements remain controlling:
- Replay is an overlay over the live picture (Option A).
- Replay is independent and must not interrupt live streaming or continuous recording.
- Access to a replay sequence must be immediate, especially for quick decision confirmation.
- Recording is continuous and saved to a user-selected location, so sequences can be retrieved even without a bookmark.
- Bookmarks are compact markers; creating one must not automatically open the bookmark drawer.
- Windows and Linux installation must be very easy.
- Production and isolated test installations must remain separate.

## 2. Findings requiring correction or clarification

### C1 — Contradictory replay-duration contract
FR-022/FR-023 require a selected replay window (for example 15 seconds) to request exactly that window. Part 5.3 also allows requesting the full archive span, and the sample loadSpan() flow builds the media URL from span.start and span.duration.

These are different contracts. The specification must choose and define the intended behavior, including:
- exact start/end calculation for a selected window;
- whether the server returns only the selected interval or a larger archive span;
- how the player seeks to the selected interval if a larger span is fetched;
- how the requested and actual durations are reported;
- how AT-023 and AT-080 verify the same contract.

Until this is resolved, acceptance results for selected replay duration are ambiguous.

### C2 — Bookmark persistence architecture is incomplete
The proposal names a server-side bookmarks.json as the source of truth, with a browser localStorage mirror, but does not define the server component/API that reads and writes the file.

A complete design must specify:
- API routes and request/response schemas;
- which process owns the file;
- atomic write/replace behavior and crash recovery;
- concurrent updates from multiple browser clients;
- validation, stable IDs, duplicate handling, and timestamp semantics;
- access control and error reporting;
- behavior when the storage device is unavailable.

A browser cannot directly write an arbitrary host file. The server-side mechanism must be explicit.

### C3 — Network binding must distinguish camera ingest from management
A localhost-only policy for UI, API, and metrics cannot be applied indiscriminately to RTMP ingest if phones on the tournament LAN publish to the PC. An RTMP listener bound only to 127.0.0.1 would not accept those remote camera connections.

Define separate exposure rules for:
- camera ingest (LAN-reachable only as required);
- browser UI;
- playback endpoints;
- management API;
- metrics;
- any remote access or reverse proxy.

Specify firewall expectations and whether authentication is required for each surface. “Localhost only” is not a complete network architecture by itself.

### C4 — MediaMTX recording filename behavior must be tested
The proposal recommends omitting .mp4 from recordPath because MediaMTX appends the extension. Rev.8 actually produced a .mp4.mp4 filename using the tested configuration.

Do not accept the recommendation as fact until a minimal isolated test confirms the exact behavior of the pinned MediaMTX v1.21.3 binary, with the exact recordPath pattern and recordFormat. Record the resulting filename and verify that the playback API can discover it.

### C5 — Bookmark post-roll and recording finalization are underspecified
If a bookmark is created at click time but the desired clip includes (for example) 15 seconds after the event, those future frames do not yet exist at click time. The specification must define:
- how the system waits for post-roll;
- how it knows the required media is available and finalized;
- whether the bookmark is immediately visible as pending;
- what happens if recording stops, the camera disconnects, or the disk fills during post-roll;
- whether the bookmark points to a time range or to a finalized media object.

This matters especially with segmented recording and fMP4.

### C6 — Two-camera synchronization target needs a real synchronization design
A shared host clock does not automatically synchronize two phones' capture timestamps, encoder pipelines, or network delays. A proposed ±500 ms target cannot be guaranteed merely by using one server clock.

Specify the timestamp source and synchronization method, then measure it using two independent phones. Define how offset is estimated, corrected, monitored, and displayed when confidence is low. Acceptance criteria should state the actual test setup and measured tolerance.

### C7 — Disk-full behavior is unrealistic as currently phrased
A promise to stop cleanly with “no partial write” is not generally guaranteed if the disk becomes full in the middle of a segment or container write.

Define observable behavior instead:
- detect low-space and write failures;
- preserve and identify completed playable segments;
- mark or quarantine incomplete/corrupt segments;
- avoid reporting recording as healthy after write failure;
- recover on restart without treating incomplete media as valid;
- specify whether recording retries, stops, or switches storage.

Test with a deliberately constrained test volume. Do not promise that a partially written file can never exist.

### C8 — Performance targets need a reference platform and measurement method
CPU, RAM, replay latency, and return-to-live thresholds are proposals, not established facts. The document should identify reference hardware, number/resolution/frame rate of cameras, browser, network, recording format, and measurement method. Distinguish hard acceptance limits from targets to be validated.

### C9 — Timestamp representation is inconsistent
The proposal describes timestamps as epoch milliseconds but also uses ISO strings such as createdAt in bookmark data. Choose a canonical representation for storage and API interchange, specify timezone handling, and define conversions at UI boundaries. Avoid ambiguous local-time strings.

### C10 — Low-disk startup policy conflicts with continuous recording
FR-016 appears to block a new recording when free space is below a critical threshold, while continuous recording is a core requirement. This needs a clear policy with retention, quota, and disk-full handling. Define whether old recordings can be deleted automatically, whether the current tournament is protected, and what happens if no safe space can be reclaimed.

### C11 — Retention and quota policy is not sufficiently precise
A quota described as a percentage of free disk space can be a moving target and may not define a stable maximum. The specification should distinguish:
- total storage allocation;
- free-space reserve;
- per-tournament retention;
- maximum age;
- automatic deletion order;
- protected/current tournament data;
- user-selected retention period.

The user has not approved a final retention policy. Do not turn proposed defaults into decisions.

### C12 — Return-to-live target needs measurement
A requirement such as closing replay and returning to live within 200 ms may be difficult to guarantee across browsers and devices. Define exactly what “returned to live” means (live element visible, first decoded live frame, or live frame presented), and measure it on the reference platform.

### C13 — Frame stepping must account for browser/media constraints
Frame-accurate stepping depends on actual encoded frame structure, keyframes, browser seeking, and decoder behavior. A nominal 1/fps seek is not proof of moving exactly one decoded frame. Specify whether the product promises exact frame stepping or best-effort stepping, and test on the actual fMP4/browser pipeline.

### C14 — The 25.001-second discrepancy remains unresolved
The browser was observed requesting duration=25.001 while the UI dropdown showed 15s. The video element later reported a duration of approximately 24.9682 seconds. The document does not explain the discrepancy.

This must remain a separate root-cause investigation. Trace the selected value through UI state, request construction, server response, and media duration. Do not mark it fixed based on an unrelated duration-contract rewrite.

### C15 — Caddy Location header rewrite remains unexplained
The current Caddyfile contains header_down Location ^/ /hls/, while the dated backup does not. The reason and effect are unknown. Test with and without the directive in the isolated test environment, capturing actual playlist and redirect behavior. Do not assume it is required or responsible for replay failure.

## 3. Additional implementation and verification cautions

- HTTP 200 from a playback endpoint does not prove that the complete media payload is valid, decodable, or presented by the browser.
- A non-empty archive list does not prove the selected interval is available or that the player can seek to it.
- canplay does not necessarily mean a requested seek has completed or that the intended frame has been presented. The design should define seek completion and frame-presentation confirmation.
- Recording health must be based on observable backend/file progress and write status, not a static UI message or merely a configured recording path.
- A stop script's process-safety checks should compare normalized executable paths with a directory boundary, not rely only on a raw string prefix.
- Cross-platform installation is a product requirement, not merely a future packaging detail. The specification should describe supported Windows and Linux distributions, prerequisites, service lifecycle, folder relocation, storage selection, firewall setup, upgrades, and recovery in plain user steps.

These are design/verification cautions, not claims that each item has already caused a runtime failure.

## 4. Required evidence before implementation approval

At minimum, the revised specification should be checked against:
1. The exact Rev.8 source/configuration package and project logs.
2. The 25.001-second browser request and the selected 15-second UI state.
3. A pinned MediaMTX v1.21.3 filename-extension test.
4. Caddy HLS behavior with and without the Location rewrite.
5. A real LAN phone-to-PC RTMP ingest test under the proposed network-binding policy.
6. Bookmark creation, post-roll completion, multiple browser clients, and storage failure behavior.
7. Two-camera synchronization using independent devices.
8. Disk-full and restart/recovery tests.
9. Actual replay pause, seek, frame stepping, zoom, and return-to-live behavior in the target browser.

Proposed acceptance tests must remain marked NOT RUN until executed. Test design is not test evidence.

## 5. Preliminary conclusion

DeepSeek's Rev.9 document is a strong starting draft, but it should not yet be approved as an implementation baseline. The most important blockers are the contradictory replay-duration contract, missing bookmark persistence API design, network-ingest/binding distinction, unverified MediaMTX filename assumption, bookmark post-roll/finalization, camera synchronization, and disk-full recovery semantics. The 25.001-second discrepancy and unexplained Caddy rewrite also remain open.

This is ChatGPT's independent review only. It does not replace DeepSeek's original analysis, and it is not yet reconciled with Gemini's forthcoming analysis. The three reviews should be compared only after Gemini's assessment is available. Preserve disagreements and source attribution rather than silently merging them.

## 6. Safety and authorization status

- No Rev.9 implementation is authorized by this review.
- No source code, scripts, configuration, service, production installation, or Rev.8 test installation was changed.
- Production D:\JudoReplayKit remains protected.
- Isolated Rev.8 test installation D:\JudoReplayKit_Rev8_Test remains protected.
- All proposed Rev.9 acceptance tests remain NOT RUN unless separately recorded as executed in TEST-LOG.md.
