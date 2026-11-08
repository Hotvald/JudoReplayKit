# Rev.9 — Three-Way Technical Review
**Date:** 2026-11-08  
**Reviewers:** DeepSeek, ChatGPT, Gemini  
**Status:** Comparative review only — Rev.9 is NOT approved for implementation.

## 1. Purpose and review boundary

This document compares the DeepSeek Rev.9 Requirements Specification, ChatGPT's independent preliminary review, and Gemini's independent technical audit. It does not merge their claims into a single assumed truth. Each statement is classified as a shared conclusion, reviewer-specific finding, hypothesis, proposal, or item requiring runtime verification.

No code, configuration, service, or installation changes are authorized by this review. The next gate is owner review of this comparison. DeepSeek must not be asked to revise Rev.9 until the owner decides whether and how it should be revised.

## 2. Executive conclusion

All three reviewers agree on the central product and engineering direction:

- Continuous recording must operate independently of replay and bookmarks.
- Replay must be an overlay and must not interrupt live streaming or recording.
- Rev.8's archive time-window logic and frontend playback state need investigation.
- Browser-local bookmark storage is insufficient as the sole durable store.
- Pause, seek, and frame-step behavior is not yet reliable enough.
- Windows/Linux portability needs an explicit, testable installation design.

The reviewers do **not** establish that a complete rewrite or a new supervisor is necessary. Nor do they establish a final retention policy, disk-full policy, camera synchronization mechanism, or browser decoding solution.

Rev.9 remains a proposal. All acceptance tests remain NOT RUN unless separately evidenced by actual test records.

## 3. Comparison matrix

| Topic | DeepSeek | ChatGPT | Gemini | Comparative assessment |
|---|---|---|---|---|
| Independent replay overlay and uninterrupted recording | Required | Required; preserve architecture boundary | Required; retain overlay and continuous recording | Strong agreement; end-to-end proof still required |
| Continuous recording without bookmarks | Required | Required | Required; cites observed file growth | Agreement; file growth does not prove recoverability or integrity |
| Archive time-window logic | Identified as a concern | Identified mismatch between query end (+1000 ms) and filter (+350 ms) | Same mismatch; also asserts five-minute lookback | Strong agreement on boundary risk; exact source version/lookback needs reconciliation |
| Frontend pause/seek/frame-step | Requires correction | canplay is insufficient; await seek/frame presentation; 1/30 fallback is approximate | Calls out missing seeked wait and approximate frame fallback | Agreement; seeked alone does not guarantee frame-accurate output |
| Bookmark persistence | Server-side persistence proposed | API, atomic writes, concurrency, validation and failure handling unspecified | SQLite or JSON/server storage proposed | Agreement on direction; implementation contract incomplete |
| Two-camera synchronization | Needs definition | Shared host clock does not synchronize independent phone capture clocks | Proposes common absolute timestamp and ±50 ms | Gemini's proposed target is not achieved merely by a shared timestamp |
| Installation portability | Cross-platform installation needs design | PowerShell and absolute paths are insufficient; explicit OS/lifecycle plan needed | Recommends compiled JudoCore supervisor | Agreement on requirement; disagreement in solution scope |
| Network security/binding | Needs separation of ingress and management interfaces | RTMP LAN ingress must not be accidentally bound only to localhost | Proposes localhost API/metrics and public RTMP/Caddy | Broad agreement; exact ports and proxy topology must be verified |
| MediaMTX filename extension | Exact version test needed | Test recordPath behavior on MediaMTX v1.21.3; .mp4.mp4 observed | Claims Rev.8 .mp4.mp4 bug and proposes corrected structure | Requires pinned-version test; do not assume path syntax |
| fMP4 arbitrary seek / decoder | Requires test | HTTP 200/canplay do not prove valid presented frame; decoder path unresolved | Hypothesizes missing moov initialization atom in byte-range retrieval | Gemini adds a hypothesis, not a confirmed cause |
| Disk-full and retention | Detailed policy needed | Avoid partial-write promises; define recovery and protected data | Proposes 30 days and delete below 10 GB | No policy approved; destructive behavior requires owner decision |
| Performance targets | Need hardware and measurement definitions | Define measurement endpoint and reference conditions | Proposes replay first frame <1 s and live delay ≤3 s | Useful candidate targets, not approved or validated |
| Health indicators | Need real health verification | File growth/backend write status, not static status | Green only with file growth in last 5 seconds | Agreement on real evidence; five-second rule is a proposal |
| Acceptance tests | AT-001–AT-081 proposed, NOT RUN | Tests need more precise observable outcomes | Six tests proposed | None should be marked passed without execution evidence |

## 4. Shared conclusions

### 4.1 Replay timing and archive discovery

The reviewers independently identify archive time-window handling as a high-priority risk. The specific logic described by ChatGPT and Gemini is:

- archive list request end boundary: now + 1000 ms;
- result acceptance filter: s.end <= now + 350 ms.

A returned span ending between those two boundaries can be rejected by the frontend even though the list query requested it. This is a strong code-level explanation for a possible “no completed recording found” condition, but runtime confirmation still requires logging the exact request bounds, returned span bounds, and filter decision from the same execution.

### 4.2 Frontend media state

All reviewers identify unreliable pause/seek/frame-step behavior. Setting currentTime is asynchronous. A canplay event does not establish that a seek has completed or that the intended frame has been presented. Waiting for seeked is an improvement, but is not by itself proof of exact frame stepping, especially with variable frame rate, keyframes, or browser decoder behavior.

### 4.3 Durable bookmarks

Browser localStorage should not be treated as the sole durable source of match events. A server-side store is a reasonable direction, but the requirements must specify API behavior, atomic persistence, concurrent clients, validation, backup/recovery, and behavior when the server or disk is unavailable.

### 4.4 Failure containment

Replay/frontend faults must not stop MediaMTX recording or the live stream. This is a product-level invariant, not merely a preferred architecture. It needs tests that observe recording file progress and live stream continuity while replay is opened, paused, sought, closed, or deliberately made to fail.

### 4.5 Portability

The current PowerShell-specific scripts and absolute Windows paths do not establish a portable Windows/Linux installation. The requirement must define supported distributions/versions, prerequisites, permissions, firewall behavior, folder relocation, storage selection, upgrades, and recovery.

## 5. Reviewer-specific findings and proposals

### 5.1 Gemini: JudoCore supervisor

Gemini recommends a new compiled supervisor, preferably Go, to generate MediaMTX/Caddy configuration, manage child processes, host a SQLite bookmark API, and monitor health.

This is a proposal, not a demonstrated necessity. It may reduce path and process-management fragility, but introduces a new component with its own lifecycle, failure modes, packaging, upgrades, and security requirements. Before approval, compare it against a simpler portable launcher/service-management design and show which requirements cannot be met without a new supervisor.

### 5.2 Gemini: five-minute archive lookback

Gemini states that openInstantReplay() uses lookFrom = now - 5 * 60 * 1000 and that this prevents discovery after a camera has been disconnected for more than five minutes.

This conflicts with ChatGPT's earlier source review, which described a 24-hour archive lookback in the Rev.8 app.js under review. This is a material source-version discrepancy. Resolve it by identifying the exact archive/package, commit or file hash, and line numbers used by each reviewer. Do not combine the two findings until version identity is established.

### 5.3 Gemini: historical HTTP test results

Gemini reports:
- a 24-hour bounded MediaMTX API query for a valid path returned HTTP 200 in 30 ms;
- a five-minute bounded query after archive end returned HTTP 404;
- an unknown path returned HTTP 400.

These are reviewer-reported historical test results. They are not independently verified by this comparative review. The exact URLs, query parameters, timestamps, logs, and runtime version should be attached to a test record before using them as acceptance evidence.

### 5.4 Gemini: missing moov atom hypothesis

Gemini suggests that a playback failure despite HTTP 200 may arise because arbitrary time-based byte-range retrieval omits the fMP4 initialization moov atom.

This is a plausible hypothesis in some MP4 delivery designs, but the supplied evidence does not establish that the Rev.8 MediaMTX playback endpoint uses the relevant byte-range behavior or that the initialization metadata is missing. Capture the actual request/response headers and payload structure, then inspect decoder errors and media initialization before selecting MSE, Canvas, or a custom parser.

### 5.5 Gemini: disk policy and targets

Gemini proposes:
- 30-day retention;
- deleting the oldest segment block below 10 GB free space;
- replay first frame within 1000 ms;
- live delay no more than 3 seconds;
- bookmark persistence within 500 ms;
- recording status green only after file growth within five seconds.

These are proposed values, not approved requirements. The deletion policy is destructive and must not be implemented without an explicit owner-approved retention and protection model. Performance targets need reference hardware, camera settings, network conditions, browser/version, and precise measurement endpoints.

## 6. Technical cautions applying to all three reviews

1. HTTP 200 proves an HTTP response was returned; it does not prove the payload is complete, decodable, or visibly presented.
2. File growth proves bytes are being written; it does not prove a segment remains playable after power loss or that all required media metadata is intact.
3. A common server UTC timeline does not synchronize independent camera capture clocks, encoders, or network paths.
4. Waiting for seeked does not guarantee frame-accurate stepping or synchronized two-camera presentation.
5. Killing a supervisor process is not necessarily equivalent to killing MediaMTX or simulating power loss. Gemini's proposed T-04 must define exactly which process is terminated and verify the intended failure mode.
6. A disk-full test must define segment closure, incomplete-file handling, protected recordings, recovery, and whether recording is allowed to stop. “Delete oldest segment block” is not sufficiently precise.
7. A five-minute versus 24-hour archive lookback must be resolved against the exact source revision before it is treated as a fact.

## 7. Priority evidence plan

Before selecting a Rev.9 architecture, run and document these investigations in an isolated test installation:

1. **Source identity:** Record exact Rev.8 package, source hashes, and configuration hashes used by each analysis. Resolve the five-minute/24-hour discrepancy.
2. **Time-boundary reproduction:** Log now, list request start/end, returned span start/end, filter thresholds, and final selection decision in one reproducible run.
3. **Playback payload:** Capture request URL, status, headers, byte ranges if any, payload structure, browser console/media errors, and first-frame presentation timing.
4. **MediaMTX filename:** Test recordPath behavior against the exact pinned MediaMTX v1.21.3 binary and record actual filenames.
5. **Caddy rewrite:** Compare behavior with and without the unexplained header_down Location ^/ /hls/ rule in an isolated environment.
6. **Pause/seek/frame-step:** Test exact observable frame/time behavior, repeated stepping, pause stability, and return-to-live. Do not equate seeked with frame accuracy.
7. **Two-camera sync:** Use two independent phones and a visible shared timing reference; measure actual offset and drift over time.
8. **Bookmark persistence:** Test refresh, browser crash, multiple clients, server restart, disk write failure, and concurrent bookmark creation.
9. **Failure containment:** Exercise replay UI failures while verifying live continuity and recording file growth.
10. **Disk-full/recovery:** Test safely on disposable storage, including incomplete segments, restart, and retention safeguards.
11. **Installation portability:** Test clean Windows and Linux machines, folder relocation, non-admin operation, firewall rules, update and rollback.

All tests are currently NOT RUN unless separate logs explicitly show otherwise.

## 8. Recommended decision sequence

1. Approve this three-way comparison as the review record, or provide corrections.
2. Resolve source/version identity and the high-priority evidence gaps.
3. Decide which Rev.9 requirements need correction or additional acceptance criteria.
4. Only then ask DeepSeek to revise the Rev.9 specification against an agreed correction list.
5. Review the revised specification before any implementation approval.

## 9. Current project status

- DeepSeek Rev.9: design proposal; not approved for implementation.
- ChatGPT review: independent preliminary review; published separately.
- Gemini audit: received 2026-11-08; compared here.
- Three-way comparison: this document.
- Implementation/configuration/service changes: not authorized.
- Retention, disk-full deletion, JudoCore architecture, authentication, and performance thresholds: open owner decisions.
- Acceptance tests: proposed or pending; not passed unless actual execution evidence is recorded.
