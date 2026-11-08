# JudoReplayKit Rev.9 — Requirements Specification (Draft 0.2)

**Date:** 2026-11-08  
**Version:** 0.2  
**Status:** DRAFT FOR OWNER REVIEW AND INDEPENDENT REVIEW — NOT APPROVED FOR IMPLEMENTATION  
**Purpose:** Define a new, independently implemented application. This is not a Rev.8 repair plan.

## Document authority and labels

- **CONFIRMED:** Explicitly established product requirement or decision.
- **DERIVED:** Necessary to make a confirmed requirement reliable and testable; review required.
- **PROPOSED:** Candidate target or design direction, not approved.
- **OPEN:** Requires explicit owner decision.
- **OBSERVED Rev.8:** Recorded symptom or source observation, not proof of root cause.
- **HYPOTHESIS:** Possible explanation not yet established.

All acceptance tests are initially NOT RUN. No proposed architecture, numeric target, retention rule, security policy or destructive behavior becomes binding without owner approval. Draft 0.2 adds explicit timing, time semantics, logical interface, health/failure, synchronization, test-evidence and traceability clarifications. Draft 0.1 remains available in Git history.

## 1. Purpose, users and operating context

JudoReplayKit is a referee-assistance application for reviewing judo match sequences while the match continues. It must let an official quickly inspect an event without interrupting live video, live streaming or continuous recording.

Primary users are judo referees and authorized tournament support personnel. The interface must be usable under time pressure, with minimal steps and clear state indication.

- **CONFIRMED:** Easy installation and operation on Windows and Linux.
- **OPEN:** Exact Windows versions, Linux distributions/versions, CPU architectures and browser support.
- **CONFIRMED:** Application UI and user-facing documentation are in English.

## 2. System-wide invariants

- **INV-01 — Live continuity (CONFIRMED):** Opening, playing, pausing, seeking, stepping, zooming or closing replay must not stop or intentionally pause live ingest or live display.
- **INV-02 — Recording continuity (CONFIRMED):** Replay operations must not stop, pause, reset or reconfigure continuous recording.
- **INV-03 — Replay independence (CONFIRMED):** Replay has its own playback position and state. Live time and replay time must not drive or overwrite one another.
- **INV-04 — Continuous archive (CONFIRMED):** Record continuously to storage selected by the user when starting a tournament. A bookmark is not required for later retrieval.
- **INV-05 — Failure containment (DERIVED):** A replay UI, decoder or playback-request failure must not crash or restart camera ingest/recording.
- **INV-06 — Honest health (DERIVED):** A component is not healthy merely because it is configured or a process exists. Health must be based on recent observable evidence.

## 3. Live video and cameras

- **DERIVED:** Support at least one live camera and a two-camera configuration.
- **OPEN:** Maximum camera count; supported phone platforms/apps; codecs, resolutions, frame rates, orientation and audio.
- **DERIVED:** Each camera has a stable logical identity independent of its network address or transient connection.
- **DERIVED:** Camera connectivity, stream ingest, recording and browser presentation are separate health dimensions.
- **CONFIRMED:** Live active indicator/button is green while the live source is healthy.
- **CONFIRMED:** Help text identifies the computer address as “PC IP”.
- **DERIVED:** “No signal” may appear only when the relevant live source is actually unavailable. A stale status timer or replay failure must not falsely cover a valid live picture.
- **OPEN:** Maximum acceptable live delay and its measurement point (camera capture to browser presentation). Rev.8's reported approximately 10-second delay is an observation, not a target.
- **DERIVED:** Camera disconnect/reconnect must be visible and recoverable without corrupting earlier recordings or silently changing camera identity.

## 4. Replay overlay and playback

### 4.1 Core behavior
- **CONFIRMED:** Replay is an overlay on top of the live picture (Option A).
- **CONFIRMED:** Replay is independently controllable and must not affect live streaming or recording.
- **CONFIRMED:** Access to a recent sequence must be immediate, especially for quick decision confirmation.
- **DERIVED:** Replay opens without navigating away from the match view. Replay and live imagery must be unmistakably distinguishable.
- **DERIVED:** Closing replay reveals the current live picture, not the live frame from when replay was opened.

### 4.2 Selected-duration contract — OPEN
Rev.8 showed a 15-second UI selection while a media request was observed with duration 25.001 seconds. Root cause remains unresolved. The Rev.9 specification must choose one consistent contract:
- **Candidate A:** Playback service returns exactly the selected interval.
- **Candidate B:** Service may return a larger containing media span, but player presents only the selected interval using explicit trim boundaries.

For either choice:
- Define interval start/end using one documented time basis.
- Expose requested interval and actual available interval separately.
- Never silently replace a selected duration with a longer or full-span replay.
- If post-roll is not yet recorded, state whether the player is waiting, showing partial footage or reporting unavailable footage.
- **OPEN:** Available duration choices and default. A 15-second default was observed in Rev.8, not reaffirmed as a Rev.9 decision.

### 4.3 Controls and playback state
- **DERIVED:** Play/pause labels and state must be truthful. Paused replay remains at its selected timestamp until resumed or sought.
- **DERIVED:** Seeking changes replay position only; it never snaps to live.
- **DERIVED:** Define frame-step semantics. Exact adjacent decoded-frame stepping may be required, but cannot be promised until the media/browser pipeline supports and passes it. A 1/30-second seek is not proof of one-frame movement.
- **DERIVED:** Distinguish loading, ready, playing, paused, ended and failed. HTTP success or media metadata alone is not proof that a visible frame was presented.
- **OPEN:** Playback speeds, audio policy and slow-motion requirement.

### 4.4 Return to live
- **DERIVED:** Provide an obvious Live action that reveals the current live frame.
- **OPEN:** Numerical return-to-live target. Measure action-to-first-presented-live-frame, not merely a UI state change.

### 4.5 Zoom
- **CONFIRMED intended behavior:** Zoom must inspect a chosen area, not be restricted to the image center.
- **DERIVED:** Zoom focal point follows the selected point/area; pan and zoom work while replay is paused.
- **OPEN:** Input methods, maximum zoom, reset behavior and whether zoom is shared across cameras.

## 5. Continuous recording and archive

- **CONFIRMED:** Continuous recording starts for a tournament and is stored in a user-selected location.
- **DERIVED:** Recording is independent of replay and bookmarks. Retrieve arbitrary intervals even when no bookmark was created.
- **DERIVED:** Recording health is based on ingest/write progress and backend errors, not a static “configured” status.
- **DERIVED:** Show per-camera recording state, destination, recent write/progress evidence, available space and errors.
- **OPEN:** Audio, codec/container, segment duration, resolution and frame-rate limits.
- **DERIVED:** Archive discovery must find older retained recordings even when the camera is currently offline. It must distinguish invalid camera/path, no data in interval, storage unavailable and internal error.
- **DERIVED:** Use explicit timestamp/time-zone semantics and handle daylight-saving changes.
- **DERIVED:** Archive index must reconcile with actual files after interrupted shutdown.
- **DERIVED:** Completed segments remain playable after normal shutdown. Incomplete segments are detectable and not reported as complete.
- **OPEN:** Search horizon/indexing approach and acceptable data-loss window after power failure. Rev.8 five-minute and 24-hour lookbacks are historical values, not product policy.

## 6. Bookmarks and match events

- **CONFIRMED:** Bookmark controls remain compact and accessible, preferably a small bottom drop-up control/drawer.
- **CONFIRMED:** Creating a bookmark must not automatically open the drawer.
- **DERIVED:** Bookmark action gives immediate unobtrusive confirmation.
- **DERIVED:** A bookmark identifies event time, camera, tournament/session and stable event ID, not merely a browser-local clip.
- **OPEN:** Default pre-roll/post-roll. Earlier discussion considered approximately 30 seconds before and 10–15 seconds after, but this is not approved.
- **DERIVED:** If post-roll is requested, bookmark may be Pending until future media is recorded/finalized; then Ready, Partial or Failed. If camera/storage fails, preserve event timestamp and report actual available footage.
- **DERIVED:** Bookmarks persist across refresh, browser restart and application restart. Browser local storage may be a cache, not the sole durable store.
- **DERIVED:** A defined application service/API owns persistence. Specify routes, schemas, validation, stable IDs, atomic writes/transactions, concurrency, backup/recovery, access control and error behavior.
- **OPEN:** Storage technology (e.g. SQLite or transactional JSON) must be chosen after review, not assumed.

## 7. Two-camera synchronization

- **DERIVED:** Two cameras share a match/tournament timeline.
- **DERIVED:** Do not assume independent phones are synchronized merely because the host PC has one UTC clock.
- **DERIVED:** Specify how capture timestamps, ingest timestamps, clock offset, encoder delay and drift are measured/estimated. Show synchronization confidence; disclose uncertainty when alignment is poor.
- **OPEN:** Required tolerance (±500 ms was discussed, not approved or proven); side-by-side versus selectable-camera replay; acceptable calibration method.
- **Acceptance:** Measure actual offset using two independent devices and a shared visible timing reference at start and after sustained recording.

## 8. Network, transport and security

Specify separately: camera ingest; browser UI; replay/media delivery; management API; metrics/diagnostics; optional remote access. A single localhost-only or all-interfaces rule is not acceptable for every service. LAN camera ingest may require LAN reachability while management/metrics need narrower exposure.

- **DERIVED:** Expose only required ports/interfaces; document firewall rules.
- **OPEN:** Authentication/pairing for camera ingest and UI; trusted-LAN-only versus credentials.
- **DERIVED:** Management and metrics must not be unintentionally exposed to the LAN/internet.
- **DERIVED:** Validate paths and reject traversal, arbitrary file access, invalid camera IDs and unauthorized archive/bookmark operations.
- **DERIVED:** Keep private keys, credentials and tokens out of public repositories, source bundles and default diagnostic exports.
- **OPEN:** RTMP, SRT, WebRTC, HLS or other transport choices. No Rev.8 protocol is mandated by this specification.
- **DERIVED:** Network loss is visible and recoverable; old archive footage must not be mislabeled as live.

## 9. User interface and usability

- **CONFIRMED:** English UI; green Live state; compact bookmark control; bookmark creation does not open drawer; help says “PC IP”.
- **DERIVED:** Primary match screen prioritizes live video and essential referee controls. Diagnostics/settings do not dominate.
- **DERIVED:** Live and replay have unmistakably different labels/states.
- **DERIVED:** Error messages explain next action and do not claim success without evidence.
- **DERIVED:** Show Camera 1 and Camera 2 status separately.
- **DERIVED:** Distinguish Connected, Live, Recording, Reconnecting, No Signal, Storage Warning, Recording Error and Replay Error where applicable.
- **OPEN:** Exact layout, colors beyond green Live, button labels, touch target sizes, minimum display dimensions and accessibility requirements.

## 10. Installation, lifecycle and updates

- **CONFIRMED:** Very easy Windows and Linux installation.
- **DERIVED acceptance requirements:** First-time installation/launch without manually editing absolute paths, YAML or Caddy config; clearly documented or selectable install location; recording location selected independently; required runtimes checked/provisioned; start/stop/status/update/diagnostics/uninstall documented for each OS; one installation cannot overwrite another; uninstall cannot silently delete recordings.
- **OPEN:** Exact supported OS releases/distributions; native installer versus portable package/container; Windows installer technology; Linux packaging; privilege requirements; service versus user-session model; update channel and rollback.
- **DERIVED:** Safe lifecycle actions affect only processes owned by this application. Validate normalized executable paths and ownership before stopping processes.
- **DERIVED:** Application files, configuration, logs, bookmarks/database and recordings have documented locations and migration/backup behavior. Moving/updating the app must not silently move/delete recordings.
- **OPEN:** Simple launcher versus dedicated supervisor. A compiled supervisor is not assumed necessary.

## 11. Storage, retention and failure handling

- **CONFIRMED:** User selects recording storage when starting a tournament.
- **DERIVED:** Validate destination writability, free space and filesystem support before recording. Show active destination and warn if unavailable.
- **OPEN:** Retention duration, quota, free-space reserve, per-tournament limits and deletion policy. Thirty days and deleting oldest recording when full have been discussed but are not approved.
- **DERIVED:** No automatic deletion until owner approves exactly what may be deleted, order, protection of current/locked tournaments and reporting.
- **DERIVED:** Detect low space/write errors; preserve finalized segments; identify/quarantine incomplete segments where possible; mark recording unhealthy; provide recovery diagnostics.
- Do not promise “no partial file can ever exist”. Define detection and recovery.
- **OPEN:** On disk full: stop, switch volume or reclaim eligible old data? This is an owner decision because it can cause data loss.
- **DERIVED:** Restart recovery reconciles index with complete, incomplete and missing files.

## 12. Logical architecture and component responsibilities

Technology-neutral responsibilities:
1. Camera ingest accepts authorized streams and reports per-camera connectivity.
2. Live delivery presents current video and reports latency/health.
3. Recording/archive writes continuous media, finalizes segments, indexes and serves time-range retrieval.
4. Replay controller/player owns replay clock, state, seek and presentation; it cannot control/reset ingest or recording.
5. Bookmark service validates/persists events and resolves them to archive intervals.
6. Application API provides authorized control/status operations.
7. UI presents live, replay, bookmarks, storage and diagnostics.
8. Launcher/lifecycle manager starts/stops only owned components if needed.
9. Diagnostics records component health/actionable errors without secrets/private media.

Required boundaries:
- Replay consumes archive/media interfaces but does not own or restart ingest/recording.
- UI does not write arbitrary host files directly.
- Bookmark persistence is mediated by a defined service/API.
- Camera ingest exposure is separate from management/metrics exposure.
- Each component defines dependencies, failure signals and recovery policy.
- Replay/UI failure must not take down recording.

**OPEN:** Media server, recording format, browser playback pipeline, backend language, database, reverse proxy, supervisor, installer and package format. Compare options against requirements, support, isolation, maintenance and test evidence. Rev.8 technologies are not automatically mandated.

## 13. Diagnostics and support

- **DERIVED:** Provide diagnostics with app/component versions, camera state, recording state, storage state, recent errors and endpoints, excluding secrets.
- **DERIVED:** Health is based on recent evidence (frames received, media presented, bytes written) with documented freshness/timeout rules.
- **DERIVED:** Logs use explicit timezone or documented UTC; event, capture, ingest and server time are not conflated.
- **DERIVED:** Diagnostic bundles exclude recordings, private keys, credentials and personal data by default.
- **DERIVED:** Stable error IDs/machine-readable codes accompany user-facing messages.

## 14. Performance and reliability

Numeric targets are not approved until a reference platform and measurement method are defined. Final specification must define: action-to-first-visible-replay-frame; camera-to-browser live delay; return-to-live; control response; recording continuity/recoverability; CPU/RAM/storage under supported camera load; camera sync offset/drift; bookmark acknowledgement; recovery after interruption.

**PROPOSED only:** First replay frame within 1.0 s on a defined warm-cache reference setup; return to live within 0.5 s; no recording interval lost because of replay actions. These targets require owner approval and measurement. Do not claim feasibility before testing.

## 15. Acceptance test plan

All tests are NOT RUN until executed with build, OS, hardware, cameras, network, browser, timestamps and evidence recorded. HTTP 200 is not proof of visible video.

**Core invariants**
- **AT-001 Live continuity:** Repeatedly open/play/pause/seek/step/zoom/close replay; verify live continues without application-induced reset.
- **AT-002 Recording continuity:** Measure file/write progress before, during and after replay; verify no replay action pauses/terminates recording and resulting archive is playable.
- **AT-003 Replay independence:** Pause replay for 60 seconds while live advances; verify replay timestamp/frame remains fixed.
- **AT-004 Return to live:** Measure action to first presented current live frame against owner-approved threshold.
- **AT-005 Failure isolation:** Cause controlled replay-only failure in disposable test; verify live/recording continue.

**Replay**
- **AT-010 Duration contract:** For every supported duration, capture UI choice, API request interval, response interval and presented interval. Verify the approved contract; repeat 10 times per duration with no unexplained substitution.
- **AT-011 Offline archive:** Stop camera publishing while retaining archive; verify latest retained recording remains discoverable and replayable.
- **AT-012 Pause stability:** Pause at known timestamp; wait 30 s; verify no movement or snap to live.
- **AT-013 Seek accuracy:** Seek to at least 10 known timestamps and verify presented interval within approved tolerance.
- **AT-014 Frame-step:** Test constant and variable frame-rate media; prove one decoded frame/action if exact stepping is promised, otherwise verify declared best-effort semantics.
- **AT-015 Zoom focal point:** Select non-central area, zoom/pan/reset while paused; verify chosen area remains focal point.
- **AT-016 No false no-signal:** Keep valid live stream while replay loads/fails; verify no false overlay covers live.

**Recording/archive**
- **AT-020 Continuous retrieval:** Record without bookmarks; later retrieve arbitrary intervals.
- **AT-021 Finalization:** Normal shutdown; verify finalized segments indexed and playable.
- **AT-022 Interrupted shutdown:** Interrupt disposable recording; verify finalized segments remain valid and incomplete media is identified.
- **AT-023 Filename/format:** On pinned runtime, verify actual filenames/extensions/container and archive discovery; prevent unexplained duplicate suffix such as .mp4.mp4.
- **AT-024 Long run:** Record for owner-approved duration; verify no silent gaps, index drift or unbounded resource growth.

**Bookmarks**
- **AT-030 UI:** Create bookmark; drawer does not open.
- **AT-031 Persistence:** Refresh, restart browser and app; verify bookmarks persist.
- **AT-032 Post-roll:** Verify Pending-to-Ready only after required media is recorded/finalized.
- **AT-033 Failure during post-roll:** Disconnect camera or storage; verify partial interval/failure state is honest.
- **AT-034 Concurrent clients:** Two clients create/edit; no silent lost update or duplicate identity.
- **AT-035 Interrupted write:** Interrupt disposable write; verify recovery and report any lost update.

**Two cameras**
- **AT-040 Alignment:** Shared visible timing event; measure offset at start.
- **AT-041 Drift:** Measure again after approved sustained duration.
- **AT-042 Camera loss:** Disconnect one camera; other continues and timeline remains usable.
- **AT-043 Honest sync:** UI discloses offset/uncertainty when outside tolerance.

**Network/security**
- **AT-050 LAN ingest:** Phone publishes under proposed binding/firewall policy.
- **AT-051 Management isolation:** Second LAN device cannot access management/metrics unless explicitly intended.
- **AT-052 Authorization:** Unauthorized client cannot change settings, bookmarks or protected controls.
- **AT-053 Path validation:** Traversal/invalid IDs rejected without arbitrary file access.
- **AT-054 Secret hygiene:** Release/diagnostic bundles contain no private keys, credentials or tokens.

**Storage**
- **AT-060 Destination validation:** Writable, read-only, missing and disconnected destination tests.
- **AT-061 Low-space warning:** Disposable storage low-space test; verify warning and status freshness.
- **AT-062 Disk full:** Fill disposable volume during segment; verify recording unhealthy, completed segments usable, incomplete data identified and recovery documented.
- **AT-063 Retention:** Only after policy approval, verify deletion order and protected tournaments.
- **AT-064 Restart recovery:** Reconcile indexes after normal and interrupted shutdown.

**Installation**
- **AT-070 Clean Windows install:** No manual path/config editing; launch and record.
- **AT-071 Clean Linux install:** Repeat for each supported distribution/version.
- **AT-072 Separate installs:** Updating/uninstalling one copy cannot alter another.
- **AT-073 Storage separation:** Record outside app directory; update app; recordings remain.
- **AT-074 Uninstall safety:** Preserve recordings/bookmarks unless explicit confirmed deletion.
- **AT-075 Least privilege:** Operate under documented normal account or specify elevation steps.
- **AT-076 Update/rollback:** Update and roll back disposable install; verify data compatibility.

**Rev.8-derived regression**
- **AT-080 Duration discrepancy:** Reproduce historical 15-second UI vs 25.001-second request with complete UI/request/response/presentation evidence; pass only after 10 consecutive correct attempts.
- **AT-081 Proxy/HLS regression:** If HLS/reverse proxy is selected, verify playlist, redirect and segment URLs. Historical Caddy header_down Location rewrite is unexplained and not automatically required.
- **AT-082 Archive-window regression:** Find recordings older than five minutes and while camera offline, within approved search policy.
- **AT-083 Decode evidence:** Verify decoded and visibly presented frames; HTTP status, MIME type, readyState or metadata alone are insufficient.

## 16. Rev.8 observations and risks

These are test inputs, not assumptions about Rev.9 root causes.

| Ref | Recorded Rev.8 observation | Rev.9 response |
|---|---|---|
| R-01 | Replay opened without visible video; “no completed recording found” reported | End-to-end archive/decode/presented-frame tests |
| R-02 | 15-second selection, 25.001-second request | One duration contract; AT-010/080 |
| R-03 | Five-minute lookback missed older archive | Archive discovery independent of camera state |
| R-04 | HTTP 200/video/mp4 did not prove visible playback | Decoder and presented-frame evidence |
| R-05 | Replay disappeared and returned after hard refresh | Long-lived session/recovery test |
| R-06 | Pause/seek/frame-step jumped to live or failed | Independent replay clock/control tests |
| R-07 | Frame-step fallback used approximate 1/30-second seek | Explicit precision semantics |
| R-08 | Zoom did not reliably follow selected area | Focal-point test |
| R-09 | Live indicator flickered/false No signal | Independent live health/presentation |
| R-10 | Bookmark opened drawer and was browser-local | Compact UI/durable persistence |
| R-11 | Duplicate .mp4.mp4 filename | Pinned-runtime filename test |
| R-12 | UI could say recording configured without write proof | Health based on actual progress |
| R-13 | Windows-only paths/scripts | Cross-platform install acceptance |
| R-14 | Wildcard listener exposure | Separate ingest/management security |
| R-15 | Private key in review archive | Secret hygiene |
| R-16 | Disk-full/interrupted-write behavior unknown | Incomplete-segment recovery |
| R-17 | Caddy Location rewrite unexplained | Do not inherit by default |
| R-18 | Two-camera synchronization unproven | Measure offset and drift |

## 17. Owner decision register

Before baseline approval, answer or explicitly defer:
1. Supported Windows versions and Linux distributions/architectures?
2. Camera platforms/apps, maximum count, resolution and frame rate?
3. Maximum live delay?
4. Replay durations and default?
5. Exact-interval response or larger-span fetch with exact trim?
6. Preloaded replay buffer and acceptable startup/storage cost?
7. Exact frame stepping or best-effort?
8. Zoom inputs and maximum?
9. Audio recording/playback or video only?
10. Bookmark pre-roll/post-roll, labels, notes, edit/delete permissions?
11. Retention period and protected data?
12. Disk full: stop, switch volume or delete eligible old recordings?
13. Two-camera sync tolerance and calibration?
14. Trusted LAN/authentication/pairing and endpoint exposure?
15. Native installer or portable package; privileges; update method?
16. Simple launcher or dedicated supervisor?
17. Reference hardware and performance targets?
18. Diagnostic export scope?
19. One active tournament per installation or multiple mats/tournaments?
20. Backup/export/import and tournament closure workflow?

## 18. Approval gates and implementation sequence

- **Gate A — Completeness:** Resolve contradictions; answer or explicitly defer owner decisions; every requirement has verification method.
- **Gate B — Independent review:** DeepSeek and Gemini review this same draft. Preserve attributed disagreements.
- **Gate C — Owner approval:** Owner explicitly approves a numbered version as implementation baseline.
- **Gate D — Architecture:** Compare viable approaches for media pipeline, archive/index, bookmarks, lifecycle/installer and network security.
- **Gate E — Plan:** Define milestones, repository structure, test strategy, data protection, build/release and rollback.
- **Gate F — Clean implementation:** Begin in a new isolated Rev.9 directory/branch. Do not overwrite production or Rev.8. Reuse Rev.8 code only after explicit review/approval.

## 19. Non-goals and prohibitions

This draft does not authorize Rev.8 repair, Rev.9 coding, PoCs, installers, generated configuration or service changes. It does not select MediaMTX, Caddy, RTMP, HLS, fMP4, SQLite, Go, a compiled supervisor or any other technology. It does not approve automatic deletion, 30-day retention, a 10 GB threshold, authentication policy or numerical performance thresholds. Proposed tests are not test evidence. Production D:\JudoReplayKit and Rev.8 test D:\JudoReplayKit_Rev8_Test remain protected.

## 20. Questions for independent reviewers

Review this exact draft independently and report:
1. Missing functional/non-functional requirements.
2. Contradictions, ambiguous language or untestable requirements.
3. Inaccurate or overly prescriptive technical claims.
4. Missing API/data contracts and failure cases.
5. Insufficient, unmeasurable or unsafe acceptance tests.
6. Missing or unnecessarily deferred owner decisions.
7. Whether the draft can become an implementation baseline after specific corrections.

Use section/requirement references. Distinguish facts, assumptions, proposals and owner decisions. Do not code, edit installations, run services or silently rewrite this document.

---
**Approval status: NOT APPROVED — independent review only.**


---

# Draft 0.2 — normative clarifications and additions

The following clauses supplement and, where explicitly stated, refine Sections 1–20 above. They are part of draft 0.2. If a reviewer finds a conflict between an earlier paragraph and this addendum, identify it explicitly; do not silently choose one interpretation. Owner approval is still required.

## A. End-to-end replay timing contract (supplements Sections 4 and 14)

The word “available” is ambiguous unless its endpoint is stated. Rev.9 shall distinguish these events in both design and test evidence:

1. **T0 — User click time:** monotonic client-side timestamp when the replay action is activated. Wall-clock/UTC context may also be logged but must not replace the monotonic measurement.
2. **Ireq — Requested sequence interval:** logical start/end timestamps requested for replay, with documented time basis and endpoint convention.
3. **Tavail — Archive availability decision:** time at which the archive service identifies which portion of Ireq is actually present and retrievable. If post-roll is not yet recorded/finalized, the wait is visible and measurable.
4. **Iapi — API/request interval:** logical interval and camera/session identifiers sent to the archive service. A request may transport a larger media span, but the transport span must be separately recorded.
5. **Imedia — Actual media/container interval:** timestamps represented by the returned media, including any decode preroll, segment padding or container duration differences.
6. **Tdecode — First decoded requested frame:** first decoded frame that belongs to the requested logical interval.
7. **Tvisible — First visibly presented requested frame:** first frame from the requested interval actually rendered in the replay surface. This is the principal user-visible replay-ready event.
8. **Tlive — Return-to-live completion:** first visibly presented current live frame after the user activates Live.

Required metrics:
- Click-to-visible replay = Tvisible − T0.
- Archive decision time = Tavail − T0.
- Decode/presentation time = Tvisible − time the relevant media becomes available to the player.
- Return-to-live = Tlive − Live-action time.

These events must be independently observable in diagnostic/test evidence. HTTP 200, a MIME type, media metadata, readyState, decoder initialization or a UI label alone does not prove Tvisible. The visible frame must be shown to belong to Ireq using media timestamps and/or identifiable reference content.

**OPEN:** Numeric thresholds. The owner’s requirement is minimal practical delay, especially for quick decision confirmation. Numeric pass/fail values require a defined reference PC, browser, cameras, network, cache condition and measurement uncertainty. The 1.0-second warm-cache replay and 0.5-second return-to-live figures in Section 14 remain proposals only.

## B. Selected-duration contract and FR-022/FR-023 legacy reference (supplements Section 4.2 and AT-010/080)

The historical Rev.8 observation was a 15-second UI selection with a media request reported as 25.001 seconds. This is a regression input, not proof of a root cause.

The product contract is about the interval the referee actually sees. The eventual design may:
- retrieve exactly the selected interval; or
- retrieve a larger containing media span and trim presentation to the selected interval.

Neither transport strategy is mandated here. In either case, the UI/request/media/presented intervals must be separately observable, and the user must never be shown a silently extended interval. Container duration alone is not sufficient proof of correct visible interval.

The earlier review reference to a conflict between FR-022/FR-023 and Part 5.3 cannot be reconciled from the fetched draft 0.1: that file contained no stable FR identifiers and Section 5 was “Continuous recording and archive”, not a Part 5.3 duration contract. Therefore the historical conflict is **unresolved as a source-traceability issue**. Do not treat the new FR-022/FR-023 labels in any later renumbered draft as necessarily identical to the old reference. The owner/reviewer should provide the original text/revision if the precise conflict is to be resolved. Section 4.2 and AT-010/080 remain the current duration-contract locations.

## C. Bookmark time semantics and mapping (supplements Section 6)

The following timestamps are different data, not synonyms:

- **Event time:** when the official activates the bookmark on the client. This records intent and may include network delay.
- **Server receipt/persistence time:** when the host receives and commits the event.
- **Camera capture time:** timestamp associated with acquisition of the frame at the camera/media source, if available and trustworthy.
- **Ingest time:** when the host receives the corresponding frame or packet.
- **Archive time:** the timeline/index coordinate used to retrieve stored media, mapped to media presentation timestamps and/or capture time.

A bookmark record must preserve the original event timestamp and its source, server receipt time, stable tournament/session and camera IDs, and per-camera mapping information. The mapping must include its method and uncertainty. If exact capture time is unavailable, store the best available estimate and disclose uncertainty; do not claim frame-exact alignment.

The bookmark service must define stable IDs, schema/versioning, validation, authorization, concurrent-write/conflict behavior, persistence acknowledgement, recovery and backup/export interfaces. This is a logical contract only: no database, file format or storage location is selected. A browser-local record may be a cache but cannot be the only durable copy.

**OPEN:** The event-time convention (client click, server receipt, selected frame or another owner-defined convention), pre-roll/post-roll, labels/notes and editing/deletion permissions. The previously discussed approximately 30-second pre-roll and 10–15-second post-roll are not approved defaults.

## D. Logical API and data contracts (supplements Sections 6 and 12)

Before implementation, each logical operation must specify:
- operation name and purpose;
- required/optional inputs, units and time basis;
- validation and authorization;
- stable output schema and schema version;
- success, partial, pending, unavailable, conflict and failure states;
- timeout/cancellation and retry/idempotency behavior;
- concurrency/version semantics;
- audit/diagnostic evidence without exposing secrets.

At minimum, logical operations are needed for:
1. camera/session discovery and health;
2. start/stop/status of a tournament recording session;
3. archive query by camera/session/time interval;
4. retrieval/streaming of the corresponding media;
5. bookmark create/read/update/delete and state;
6. storage destination validation and storage health;
7. application/component health and diagnostics.

These are capability contracts, not a mandate for REST, a particular route naming convention, process topology, language or database. A replay request must not have side effects on recording state. Archive query failure must not mutate or invalidate recording data.

## E. Health model and evidence (supplements Sections 3, 5, 9 and 13)

The following are separate health dimensions and must not be collapsed into a single green lamp:

- **Camera connected:** transport/session is connected.
- **Frames arriving:** recent frames/packets are received.
- **Live frames presented:** current frames are visibly rendered in the live surface.
- **Recording writes progressing:** actual media bytes/segments are being written and finalized as expected.
- **Archive query available:** the archive service can answer a valid query.
- **Replay decoded:** decoder has produced a frame in the requested interval.
- **Replay visibly presented:** that frame is actually visible in the replay surface.
- **Storage healthy:** destination is writable and within the approved storage policy.

Each state includes evidence source, last-success time, freshness/timeout rule and error. A connected camera does not prove frames are arriving; arriving frames do not prove live presentation; an open recording file does not prove writes are progressing; successful HTTP does not prove visible replay. The UI must show the relevant dimension and avoid claiming more than the evidence establishes.

## F. Failure containment and recovery (supplements Sections 2, 12 and 13)

For each logical component, the architecture/operations design must state dependencies, failure signals, containment boundary, recovery action, retry/backoff behavior, state reconciliation, user-facing status and evidence retained.

Minimum containment expectations:
- Replay/player/decoder failure cannot stop or restart ingest or recording.
- One camera failure cannot stop the other camera.
- Archive query/index failure cannot delete or rewrite media.
- UI failure cannot stop recording.
- Storage write failure must change recording health explicitly; completed segments are preserved where possible and incomplete segments identified.
- Recovery must not repeatedly restart a healthy component or silently present missing intervals as continuous footage.

Recovery timing and retry values remain open until the reference platform and failure model are defined.

## G. Two-camera synchronization measurement (supplements Section 7 and AT-040–043)

A shared UTC clock is not proof of frame alignment. The synchronization assessment must distinguish:
- camera clock offset;
- clock drift over elapsed time;
- capture timestamp accuracy;
- ingest delay;
- encoder/transport delay where measurable;
- actual corresponding-frame offset;
- measurement uncertainty/confidence.

The test must use a defined common reference event visible to both cameras, document how corresponding frames are selected, measure offset at the start and after an owner-approved sustained duration, and report uncertainty. A manual marker is only a calibration aid unless the system proves that it is associated with the actual captured frame/time on each camera.

**OPEN:** Required tolerance (±500 ms was discussed but is not approved), calibration procedure and UI presentation of uncertain alignment.

## H. Frame-step capability and UI truthfulness (supplements Section 4.3 and AT-014)

The chosen implementation must declare one of these capability levels after technical evaluation:
- **Exact decoded-frame step:** each action presents the immediately adjacent decoded frame in the requested direction, with documented behavior for variable frame rate and keyframe boundaries.
- **Best-effort temporal step:** action seeks to a defined target time; the actual presented frame may differ, and UI/help text must state this accurately.

A best-effort label is not a substitute for a functioning control. Tests must verify actual frame/timestamp movement, direction, repeatability and behavior at start/end of media. The capability level remains OPEN until the media/browser pipeline and support matrix are selected and tested.

## I. Test evidence and reference environment (supplements Section 15)

Every executed acceptance-test record must include:
- Test ID and requirement IDs covered.
- Build/version and cryptographic hash.
- Date/time/timezone, operator and test environment identifier.
- PC hardware, OS, browser and relevant driver versions.
- Camera make/model/app, resolution/frame rate and encoding settings.
- Network topology and measured conditions.
- Storage device/filesystem and free-space state.
- Preconditions and exact reproducible steps.
- Expected and observed results.
- Measurement method, raw values and uncertainty.
- Logs, request/response traces, decoded-frame/presentation evidence and relevant screenshots/screen recording.
- Pass, Fail, Blocked or Not Run status, with deviations.

For tests of visible replay, include proof of decoded and visibly presented frames from the requested interval. HTTP status, Content-Type, Content-Length, readyState or container duration alone is insufficient. For recording, include actual write progression and later archive/decode verification. For sync, include reference event, offset/drift calculation and uncertainty.

Before numeric targets are binding, define a reference profile: PC CPU/RAM/GPU/storage, OS, browser, camera devices/settings, number of cameras/clients, network path/conditions, storage filesystem, cache condition, clock source and instrumentation. Until then, timed tests may be executed exploratorily but cannot be declared passed against invented thresholds.

## J. Additional explicit acceptance checks

The existing AT catalogue remains in force. The following clarify required evidence and add checks:

- **AT-084 — Replay timing trace:** Record T0, Ireq, Tavail, Iapi, Imedia, Tdecode and Tvisible for repeated replay actions. Pass only when all events are distinguishable and visible presentation is tied to the requested interval.
- **AT-085 — Bookmark time mapping:** Create bookmarks from a known common event; preserve client event time, server receipt time and per-camera mapping/uncertainty. Pass only when retrieved frames correspond within an owner-approved tolerance or uncertainty is honestly reported.
- **AT-086 — Health-signal independence:** Independently simulate connected/no-frames, frames/no-presentation, recording configured/no-write, archive unavailable and replay decode failure. Pass only when each dimension reports its own evidence and no unrelated green/failed state is inferred.
- **AT-087 — API concurrency and idempotency:** Repeat a bookmark create request and perform concurrent updates from two clients. Pass only when stable identity, duplicate behavior and conflict handling match the documented contract.
- **AT-088 — Component failure containment matrix:** In a disposable environment, trigger one fault at a time in UI, replay, archive, one camera and storage. Pass only when unaffected components remain healthy and recovery/reporting matches the documented boundary.
- **AT-089 — Synchronization uncertainty:** Perform start/end reference-event measurements and compare displayed confidence/uncertainty with measured offset/drift. Pass only when the UI does not claim alignment beyond the evidence.
- **AT-090 — Frame-step truthfulness:** Verify the declared exact or best-effort capability using decoded frame identities/timestamps. Pass only when observed behavior and UI description agree.

## K. Open decisions retained — no silent defaults

The following remain owner decisions or architecture investigations, not approved choices:
- retention duration, quota, low-space reserve and deletion eligibility/order;
- disk-full response (stop, switch destination or delete approved eligible data);
- Windows/Linux support matrix and CPU architectures;
- camera platforms, camera count, resolutions and frame rates;
- live delay, replay duration/default and numeric timing thresholds;
- exact-range versus larger-span retrieval/trim;
- preload/cache strategy and resource cost;
- frame-step precision level;
- zoom behavior and inputs;
- audio policy;
- bookmark pre-roll/post-roll, event-time convention and permissions;
- synchronization tolerance/calibration;
- authentication, pairing and network exposure per endpoint;
- transport, codec/container and segment duration;
- installer/package, privilege model, update/rollback;
- simple launcher versus supervisor;
- language, database, process topology and persistence format;
- backup/export/import and diagnostic privacy scope.

No reviewer proposal (including 30-day retention, 10 GB free-space threshold, automatic oldest-first deletion, specific ports, localhost/all-interface binding, a particular database, protocol, codec or supervisor) is an owner decision.

## L. Version 0.2 change summary

Compared with draft 0.1, this revision:
1. Adds an explicit end-to-end replay timing model from user click through first visibly presented frame and return-to-live.
2. Clarifies that the user-visible interval, not container duration or request size alone, defines correct replay.
3. Separates bookmark event time, server receipt, camera capture, ingest and archive time, including mapping uncertainty.
4. Defines logical API/data-contract expectations without selecting routes, language, database or process topology.
5. Expands health into independent evidence-based dimensions.
6. Requires component-by-component failure containment and observable recovery.
7. Strengthens two-camera synchronization measurement to include offset, drift, actual frame alignment and uncertainty.
8. Clarifies exact versus best-effort frame stepping and prohibits a label from replacing a functioning control.
9. Adds reference-environment and evidence requirements for acceptance tests.
10. Adds AT-084 through AT-090 for timing trace, bookmark mapping, health independence, API concurrency, failure containment, sync uncertainty and frame-step truthfulness.
11. Explicitly records the legacy FR-022/FR-023 versus Part 5.3 reference as unresolved because the cited identifiers/section were not present in the fetched draft 0.1.
12. Keeps all unapproved technology, numeric targets, retention and destructive storage policies open.

**Draft 0.2 approval status: NOT APPROVED — owner review and independent review only.**
