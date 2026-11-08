# JudoReplayKit Rev.9 — Requirements Specification

**Date:** 2026-11-08  
**Version:** 0.4  
**Status:** DRAFT 0.4 — FINAL OWNER-APPROVAL CANDIDATE; NOT APPROVED FOR IMPLEMENTATION  
**Purpose:** Define the product requirements and acceptance basis for a clean, independent Rev.9 implementation. Rev.9 is not a repair or continuation of Rev.8 architecture.

## Document authority and interpretation

Requirement labels:
- **CONFIRMED** — explicitly decided by the owner.
- **DERIVED** — necessary to make a confirmed requirement reliable, safe or testable; reviewers should challenge it if excessive.
- **PROPOSED** — candidate approach, not binding.
- **OPEN** — requires later owner decision, feasibility study or reference-environment test.
- **OBSERVED Rev.8** — historical symptom/evidence only; not proof of cause and not a Rev.9 design instruction.

This draft incorporates the owner decisions recorded in `DECISIONS.md` through 2026-11-08 and the project history in `PROJECT-LOG.md`. Where an earlier note conflicts with a later explicit owner clarification, the later clarification controls. Bookmarks are markers, not fixed-length clips; the earlier 30-second pre-roll/15-second post-roll idea is withdrawn. Draft 0.4 incorporates the owner's final clarifications on live-edge replay, two-camera bookmarks, independent per-camera frame stepping, slow motion, Live Delay and safe storage-full behavior.

All acceptance tests are initially **NOT RUN**. No architecture, numeric performance threshold, security default, protocol, data format or implementation technology is approved by this draft. No implementation may begin until the owner approves a numbered final specification.

## 1. Product purpose and priority

JudoReplayKit helps judo referees inspect techniques and match events while the match continues. It must not interrupt the live feed, stream ingest or continuous recording.

**Primary product goal:** A simple, reliable, low-friction workstation for the referee at one mat, with two camera views, selected-delay live viewing, immediate Instant Replay and continuous recording.

**Priority order:**
1. Per-mat referee workflow and reliability.
2. Two simultaneous camera feeds, continuous recording and usable archive/replay.
3. Easy local installation and operation on Windows and Linux.
4. Accurate state, diagnostics, recovery and safe storage behavior.
5. Optional enhancements only where they do not add material complexity or compromise priorities 1–4.

**CONFIRMED:** Application UI and user-facing documentation are in English. Development discussion may be in Danish.

## 2. Deployment model and equipment

### 2.1 Baseline per mat

**CONFIRMED owner deployment assumption — each mat has:**
- One PC running Windows or Linux.
- One local router/network.
- Two mobile phones or other streaming cameras.
- Two camera tripods.
- A power strip and/or extension cable reel with outlets.
- One wireless mouse.

The two cameras are positioned in **opposite corners of the competition area** to capture the athletes' techniques from two different angles. The two camera positions should provide complementary, useful views of the contest area.

The software must support **two simultaneous camera streams per mat** as a core baseline, not as a paid or optional add-on. The exact camera models, height, lens, field of view, orientation, mounting position and network settings must be validated during practical setup tests. Do not assume compatibility solely from a camera brand name.

### 2.2 Operating-system scope

- **CONFIRMED:** Easy installation and ordinary operation on Windows and Linux are product requirements.
- **OPEN:** Exact supported Windows releases, Linux distributions/releases, CPU architectures and browser versions. These must be stated before release and tested on clean installations.
- **CONDITIONAL / NOT PROMISED:** ChromeOS is desirable only if support is not disproportionately difficult. It is an exploratory feasibility item and must not delay Windows/Linux or be advertised as supported until validated.
- Browser-based UI is acceptable, but browser/runtime requirements and limitations must be documented.

### 2.3 Mats and supervisor feature

**CONFIRMED:** The primary baseline is independent operation at each mat.

**CONFIRMED:** A supervisor feature for switching between mats and viewing their live feeds/Instant Replay is strictly **nice-to-have**. It must not complicate the project, delay the core product or compromise the per-mat workflow. It may be deferred or omitted from the initial release if it adds material complexity.

Multi-mat supervisor management is not required for the initial release. Any later consideration is optional and must be additive, low-complexity and must not create shared replay/recording bottlenecks or cross-mat failure coupling.

## 3. Core system invariants

- **INV-01 — Live continuity (CONFIRMED):** Replay operations must not pause, stop, reset or intentionally interrupt camera ingest or live viewing.
- **INV-02 — Recording continuity (CONFIRMED):** Replay operations must not pause, stop, reset or reconfigure continuous recording.
- **INV-03 — Replay independence (CONFIRMED):** Replay has its own clock, position and state; it cannot drive or overwrite live time.
- **INV-04 — Continuous archive (CONFIRMED):** Record continuously to the storage location selected by the user when starting a tournament. Bookmarks are not required to retrieve later footage.
- **INV-05 — Two-camera baseline (CONFIRMED):** Two camera feeds can be received and recorded simultaneously for one mat.
- **INV-06 — Local core operation (CONFIRMED):** Core use is local; after installation, live ingest, local recording, archive, replay and bookmarks must not require internet access, account registration, cloud relay or subscription.
- **INV-07 — No mandatory paid dependency (CONFIRMED):** No subscription, paid cloud service, paid plugin, paid third-party component/license or premium streaming-tool tier may be required for the core product to function optimally.
- **INV-08 — Failure containment (DERIVED):** A UI, replay or decoder failure must not crash or restart camera ingest or continuous recording.
- **INV-09 — Honest health (DERIVED):** Configured processes or HTTP success alone do not prove a camera is delivering frames, recording is writing or replay is visibly working.
- **INV-10 — Non-destructive storage (CONFIRMED):** The application must never automatically delete recordings to free space. Any deletion requires explicit user approval of the exact deletion scope.

## 4. Live video, camera ingest and selected delay

### 4.1 Camera and live behavior

- **CONFIRMED:** The operator must be able to see the live camera feed with a user-selected delay, separately from Instant Replay.
- **CONFIRMED:** Two cameras are supported simultaneously per mat.
- **CONFIRMED:** Live active indicator/button is green while the relevant live source is healthy.
- **CONFIRMED:** Help text refers to the computer address as **“PC IP”**.
- **DERIVED:** Camera 1 and Camera 2 have separate connection, frame-arrival, live-presentation and recording states.
- **DERIVED:** “No signal” may appear only when that camera's live source is genuinely unavailable. A replay error or stale UI timer must not falsely cover a valid live picture.
- **DERIVED:** Live and replay images/states must be unmistakably distinguishable.
- **CONFIRMED:** Live Delay choices are **1, 2, 3, 4 and 5 seconds**, with **2 seconds as the default**. Live Delay is independent of Instant Replay look-back.
- **DERIVED:** When both delayed camera views are shown, align them using the measured per-camera timing model and communicate offset/uncertainty honestly; do not claim unsupported synchronization accuracy.
- **OPEN:** Maximum acceptable live delay and measurement point (camera capture to visible PC presentation). Establish numeric targets only on a defined reference setup.
- **OPEN:** Supported camera/mobile platforms, source apps, resolutions, frame rates, orientations, codecs and audio.

### 4.2 Interoperability and free streaming tools

- **CONFIRMED:** The system must accept streams from free tools such as PRISM Live Studio using free functionality. PRISM runs on the mobile device/camera sending video; it is not a required desktop host application.
- **CONFIRMED:** OBS Studio is a relevant free/open-source interoperability and test target on Windows and Linux.
- **DERIVED:** Define a protocol-based, documented ingest contract rather than coupling the core to one named streaming app.
- **DERIVED:** Test PRISM's free Custom RTMP capability on a platform where PRISM supports it. Do not claim PRISM desktop support on Linux; current official desktop availability has been Windows/macOS, not Linux.
- **DERIVED:** Test OBS as a free source on supported Windows and Linux host platforms.
- **CONFIRMED:** Optional paid features in external applications are not required; no PRISM Plus or other premium tier may be a JudoReplayKit dependency.
- **OPEN:** Final protocol matrix and camera-specific compatibility list. Protocol choice is an implementation decision after compatibility and performance testing.

## 5. Instant Replay

### 5.1 Presentation and independence

- **CONFIRMED:** Replay is an overlay on top of the live picture (Option A).
- **CONFIRMED:** Replay is independently controlled and must not affect live ingest, live display or continuous recording.
- **CONFIRMED:** Access to a recent sequence must be immediate, especially for quick confirmation while the match continues.
- **DERIVED:** Replay opens without navigating away from the match view. Closing replay returns to the **current** live picture, not the frame that was live when replay opened.
- **DERIVED:** Replay must have independent play, pause, seek, frame-step and zoom behavior. A paused replay must remain paused at its selected media position while live continues.

### 5.2 Selected look-back interval

**CONFIRMED owner decision:** Available look-back choices are **5, 10, 15, 20 and 30 seconds**, with **15 seconds as the default**.

The selected duration is the user-visible interval immediately preceding the instant the Replay action is pressed. It ends at the Replay press time.

Example: pressing Replay at 14:32:10 with 15 seconds selected means the requested visible interval is 14:31:55–14:32:10.

Requirements:
- The selected duration defines the logical interval the referee sees; it does not prescribe the internal API request, container span or retrieval implementation.
- Distinguish the requested logical interval, available archive interval, internal request interval, returned media/container interval and actually presented frames.
- A larger internal media span is acceptable only if the player trims/presents the requested logical interval accurately and does not silently expose an unintended longer interval.
- If part of the interval is not yet available, report the actual condition clearly; do not silently substitute a different duration.
- **CONFIRMED live-edge behavior:** Replay opens immediately and must not wait for an in-progress recording segment to be finalized. If the complete selected interval is not yet available at the Replay press time, show the footage/interval that is actually available and clearly disclose that it is partial. Do not claim the full selected interval is present.
- **OPEN implementation choice:** The mechanism for accessing in-progress/rolling-buffer media, and exact-range retrieval versus a larger containing span followed by trim, remain implementation choices. The user-visible immediate/partial-availability behavior is binding.

### 5.3 Playback controls and exact frame stepping

- **CONFIRMED owner decision:** Frame-by-frame means exactly one actual decoded frame forward or backward per action, not an approximate time jump.
- **DERIVED:** Do not label an approximate seek as frame-by-frame. UI must not claim exactness unless the media pipeline demonstrably supports it on the supported media/browser matrix.
- **CONFIRMED:** The operator can select Camera 1 or Camera 2 and inspect that camera's recording independently, frame by frame. Each action moves exactly one actual decoded frame forward or backward in the selected camera.
- **CONFIRMED clarification:** The two cameras do not need to display identical frames or remain frame-identical in time/position during frame-by-frame inspection. The priority is exact independent inspection of either recording.
- **DERIVED:** Play/pause labels and states must be truthful. Paused replay cannot drift or snap to live. Seeking and frame stepping affect replay only.
- **OPEN feasibility gate:** Prototype and verify exact decoded-frame stepping for each supported camera/media/browser combination, including supported frame rates and variable-frame-rate behavior. If exact stepping cannot be reliably delivered, report the limitation and obtain owner approval before weakening the requirement. Do not silently downgrade to best-effort.
- **CONFIRMED:** Instant Replay supports slow-motion playback as well as normal playback, pause, seek and frame stepping. Slow motion must not affect live viewing, Live Delay, ingest or continuous recording.
- **OPEN:** Exact slow-motion speed choices and audio behavior; determine through feasibility and practical testing and document the actual supported behavior.

### 5.4 Zoom and return to live

- **CONFIRMED intended behavior:** Zoom must inspect a user-selected area, not only the image center.
- **DERIVED:** Selected focal point remains the zoom focus; pan/zoom must work while replay is paused.
- **OPEN:** Input methods, zoom range, reset behavior and whether zoom is independent per camera.
- **CONFIRMED:** A clear Live action returns to the current live picture.
- **OPEN:** Numeric return-to-live threshold, to be established through reference-hardware testing.

## 6. Continuous recording, archive and tournament storage

- **CONFIRMED:** Continuous recording starts for a tournament and uses a storage location selected by the user.
- **CONFIRMED:** Both camera feeds are recorded continuously.
- **CONFIRMED:** Footage must remain retrievable even when no bookmark was created.
- **DERIVED:** Show selected destination, per-camera recording status, recent write progress, free space and storage errors.
- **DERIVED:** Archive search works for retained footage even if the camera is offline.
- **DERIVED:** Archive query distinguishes at least: valid media found; no media in requested interval; invalid camera/session/path; storage unavailable; incomplete/corrupt media; and internal error.
- **DERIVED:** Define segment states such as in-progress, finalized, incomplete and corrupt. Do not present incomplete media as finalized/complete.
- **DERIVED:** Reconcile the archive index against actual files after interrupted shutdown or restart.
- **OPEN:** Codec/container, segment duration, resolution/frame-rate limits, indexing strategy, search horizon and acceptable power-failure data-loss window.
- **DERIVED:** Camera reconnect must preserve prior media and identity and must create a clearly identifiable new recording segment/session boundary where needed; do not silently append incompatible timelines.
- **DERIVED:** Time data must distinguish camera capture/event time, ingest time, server receipt/persistence time and archive time. Document UTC/local display and daylight-saving behavior.

## 7. Bookmarks and event markers

**CONFIRMED owner decision:** A bookmark is a **marker/pointer to a moment in the continuous recording**, primarily for later learning and review. It is not a clip and does not trigger fixed pre-roll or post-roll generation.

- **CONFIRMED:** Bookmarks can be created from Instant Replay.
- **CONFIRMED:** The bookmark points to the media/event timestamp currently being viewed, not the wall-clock time when the bookmark button is pressed.
- **CONFIRMED:** One bookmark represents one shared event/reference moment and stores that shared event timestamp plus the corresponding mapped media position for each camera.
- **CONFIRMED:** Opening a bookmark should show both camera views wherever each camera's recording covers the event. The views need not be perfectly identical in time; per-camera mapping and uncertainty must be preserved honestly.
- **CONFIRMED:** The continuous recording is the source of truth; later, the user can jump to the marker and inspect surrounding footage as desired.
- **CONFIRMED:** No automatic 30-second pre-roll/15-second post-roll. That earlier proposal is withdrawn.
- **CONFIRMED:** Compact bookmark control, preferably a small bottom drop-up/drawer; creating a bookmark must not automatically open the drawer.
- **OPEN / PRESERVE:** Live-time bookmarking has not been withdrawn. Retain it unless a later owner decision changes it; when used, it must map to the media/event time represented by the live view and its selected delay.
- **DERIVED:** A marker has a stable ID and identifies tournament/session, mat, camera or camera selection, event/media timestamp, creation time and any timestamp-mapping uncertainty needed to find the correct archive position.
- **DERIVED:** Show immediate unobtrusive success/failure confirmation. A bookmark must not claim success if persistence failed.
- **DERIVED:** Markers persist across browser refresh, browser restart and application restart. Browser-only transient storage must not be the sole durable record.
- **DERIVED:** Duplicate clicks/retries must not create ambiguous duplicate markers unless the user intentionally asks for duplicates. Concurrency/conflict behavior must be documented.
- **OPEN:** User-editable labels/notes, export/import, permissions and backup format.

## 8. Two-camera alignment and display

- **CONFIRMED:** Automatic alignment of the two cameras is desired, with honest indication of measured offset/uncertainty.
- **DERIVED:** Do not claim perfect synchronization merely because both streams use the PC/server clock.
- **DERIVED:** Separate capture-time estimate, ingest-time observation, clock offset, encoder/transport delay, drift and actual observed frame alignment where measurable.
- **DERIVED:** UI communicates states such as aligned within measured uncertainty, approximate/uncertain, or unknown/unavailable. Do not invent numeric accuracy guarantees.
- **OPEN:** Calibration/reference method, acceptable alignment tolerance and whether replay defaults to side-by-side or selectable camera view.
- **Acceptance:** Measure offset at start and after sustained recording with two independent cameras and a repeatable shared visible timing event. Report measured offset, drift and uncertainty. A clapper/flash at ingest does not by itself prove camera capture-time synchronization unless capture-time mapping is established.

## 9. Network access and security

**CONFIRMED owner preference:** Straightforward local access by default, with optional password protection, provided usability is not undermined.

Requirements:
- **DERIVED:** Separate camera ingest, browser UI, replay/media delivery, management/control API and diagnostics/metrics exposure.
- **DERIVED:** Only expose required services/interfaces; document ports and firewall behavior.
- **DERIVED:** Do not expose management/metrics or arbitrary file access unintentionally to the LAN or internet.
- **DERIVED:** Validate camera IDs, paths, session IDs and archive requests; reject traversal and unauthorized operations.
- **DERIVED:** Do not include credentials, tokens or private keys in repositories or diagnostic exports.
- **OPEN:** Exact default binding/exposure, optional-password activation flow, camera pairing/authentication, session timeout and remote-access policy. These require security review while preserving low-friction local operation.
- **CONFIRMED:** Internet is not required for local core operation after installation. External public streaming destinations, if the user independently chooses them, are outside the local core requirement and may have their own costs/accounts.

## 10. Storage safety and failure handling

- **CONFIRMED:** No automatic deletion, including oldest-first cleanup, is allowed.
- **CONFIRMED:** Any deletion requires explicit user approval before files are removed, with the exact selection/scope clearly shown.
- **DERIVED:** Warn about low storage and destination unavailability before data loss occurs where detectable.
- **DERIVED:** If the destination is removed, full or unwritable, stop claiming recording is healthy; show an actionable warning and preserve already finalized files.
- **DERIVED:** Do not silently switch to another destination or delete old footage.
- **CONFIRMED:** Warn about low storage and stop recording in a controlled manner before the destination is completely exhausted. Establish the safety reserve through practical tests on the supported storage/reference setup; no fixed reserve value is assumed.
- **CONFIRMED:** Preserve existing recordings. Never automatically delete recordings or silently switch destinations. Any deletion requires explicit user approval of the exact scope.
- **DERIVED:** A controlled stop must leave completed recordings accessible and classify any incomplete tail segment honestly. The UI must clearly distinguish recording stopped for storage safety from healthy active recording.
- **DERIVED:** Detect incomplete/corrupt tail segments where possible and exclude them from normal completed-archive results while preserving diagnostic evidence.
- **DERIVED:** Replay/read I/O must not starve recording writes, but scheduling must also avoid indefinite replay starvation. Use bounded resource contention and test under worst-case simultaneous two-camera write plus replay/read load.
- **Safety:** Disk-full, removable-storage and destructive deletion tests must use disposable virtual/test volumes or explicitly designated test data, never the host system disk or real tournament recordings.

## 11. Health, diagnostics and recovery

- **DERIVED:** Health is multidimensional and based on recent evidence, not process existence alone:
  - camera connected;
  - frames arriving;
  - live frames visibly presented;
  - recording writes progressing;
  - archive query available;
  - replay decoded;
  - replay frame visibly presented;
  - storage healthy;
  - camera alignment confidence.
- **DERIVED:** Define freshness/timeout and stale/unknown states for each signal.
- **DERIVED:** Distinguish ready, loading, playing, paused, ended and failed replay states.
- **DERIVED:** User-facing errors identify the affected component and next action. Do not claim success before evidence exists.
- **DERIVED:** Logs have bounded growth/rotation, explicit timezone conventions and stable error identifiers.
- **DERIVED:** Diagnostic bundle excludes recordings, credentials, private keys and personal data by default; user reviews contents before sharing.
- **OPEN:** Log retention duration/size, diagnostic export format and user privacy controls.
- **DERIVED:** Recovery behavior is defined for camera disconnect/reconnect, application crash/restart, PC reboot, router/network loss, storage removal and partial segment writes. A recovered app must not mislabel old footage as live.

## 12. Installation, lifecycle and licensing

- **CONFIRMED:** Easy installation and operation on Windows and Linux.
- **DERIVED acceptance requirements:** Ordinary first-time setup must not require editing absolute paths, YAML, reverse-proxy configuration or command-line scripts. Recording destination is selected through a clear workflow. Provide clear start/stop/status, update/rollback and uninstall procedures for each supported OS.
- **DERIVED:** Uninstall/update must not silently delete or move tournament recordings or bookmarks.
- **DERIVED:** App lifecycle actions may stop only processes/services owned by this application; no indiscriminate process-name killing.
- **OPEN:** Native installer/portable package/container, service versus user-session process, privileges, update channel and rollback technology.
- **CONFIRMED:** All bundled third-party components must have license/redistribution terms compatible with the intended distribution. No mandatory paid commercial license.
- **DERIVED:** Maintain a dependency inventory with name, version, source, license, redistribution obligations and update/security responsibility. Complete legal/license review before release.

## 13. Technology-neutral component boundaries

The following are responsibilities, not a mandated architecture or technology stack:
1. Camera ingest and per-camera connection/frame health.
2. Live delivery with selected delay and visible live-state evidence.
3. Continuous recording and archive/indexing.
4. Independent replay controller/player with exact frame-step capability.
5. Bookmark/event-marker persistence and archive resolution.
6. Local control/status interface with endpoint authorization.
7. User interface for live, replay, bookmarks, storage and diagnostics.
8. Optional lifecycle/launcher component if required.
9. Diagnostics and recovery reporting.

Boundaries:
- Replay consumes archive/media services but cannot own/restart ingest or recording.
- UI must not have arbitrary direct host-file access.
- Bookmark writes pass through a durable validated service boundary.
- Management/metrics exposure is separate from camera ingest.
- Optional supervisor/multi-mat functionality must not be a dependency of per-mat operation.

**OPEN:** Backend language, database, media server, protocol, codec/container, browser pipeline, reverse proxy, installer technology, process topology and preload/cache strategy. Evaluate against functional requirements, cross-platform support, license, maintenance, resource use and test evidence. Rev.8 components are not automatically mandated.

## 14. Performance and reliability

**CONFIRMED functional expectations:**
- Immediate access to selected recent footage.
- Replay controls do not affect live or recording.
- Straightforward return to current live.
- Continuous recording under supported two-camera load.
- Exact one-decoded-frame stepping if delivered as required by Section 5.3.

**OPEN:** Numeric thresholds for replay click-to-visible, selected live delay, return-to-live, seek, bookmark acknowledgement, camera sync, recovery and sustained resource load. Establish after selecting reference hardware, cameras, network, storage, OS and browser, then obtain owner approval before making numbers binding.

Do not convert subjective words such as “immediate” into invented numeric pass/fail thresholds. Record exploratory measurements first. A numeric target is binding only after owner approval and a repeatable measurement procedure.

## 15. Acceptance and validation plan

All tests start **NOT RUN**. A test is not passed until run against a Rev.9 candidate and its build hash, OS, PC, browser, camera models/settings, network, storage and evidence are recorded. HTTP 200, Content-Type, file size or media readyState alone do not prove visible playback.

### Core workflow and invariants
- **AT-001 Live continuity:** Open/play/pause/seek/step/zoom/close replay repeatedly; live feed remains active and current.
- **AT-002 Recording continuity:** Verify both camera writes progress before/during/after replay; archive remains decodable.
- **AT-003 Replay independence:** Pause replay while live advances; replay position remains fixed.
- **AT-004 Return to live:** Verify Live returns to the current presented live frame; record latency for later numeric target setting.
- **AT-005 Failure isolation:** Inject replay-only failure in a disposable environment; ingest and recording continue.

### Two-camera and live delay
- **AT-010 Two-camera ingest:** Send two simultaneous streams; verify correct identity, live presentation and no cross-camera mix-up.
- **AT-011 Two-camera recording:** Record both feeds continuously and retrieve each independently after source disconnect.
- **AT-012 Selected live delay:** Verify choices 1/2/3/4/5 seconds and 2-second default against measured camera-to-visible presentation; record actual delay and repeatability.
- **AT-017 Dual-camera live-delay alignment:** With a repeatable shared visible event, verify both delayed views use the measured per-camera timing model and that displayed offset/uncertainty is honest; no unapproved numeric tolerance is assumed.
- **AT-013 Camera reconnect:** Disconnect/reconnect one camera; preserve old recording, identify new session/segment and recover without affecting the other camera.
- **AT-014 Camera position/setup:** With tripods in opposite corners, verify both views show useful complementary coverage of the competition area; document camera placement/settings.

### Instant Replay
- **AT-020 Look-back interval:** Test 5/10/15/20/30 seconds, including default 15 seconds. Capture click time, logical interval, API/request span, media span and actually visible frames. Verify exact logical user interval.
- **AT-027 Slow motion:** Verify slow-motion playback at every speed offered by the product, including stable visible progression and truthful speed indication. Confirm pause/seek/frame-step interaction and that live display, Live Delay, ingest and both recordings remain unaffected.
- **AT-090 Live-edge replay availability:** Press Replay while the newest recording segment is still in progress. Verify replay opens without waiting for finalization; if the full look-back interval is unavailable, verify the actual available interval is shown and no missing footage is implied.
- **AT-021 Pause stability:** Pause at a known media timestamp; wait 30 seconds; verify no drift or snap to live.
- **AT-022 Seek accuracy:** Seek to known timestamps and verify actual visible frames correspond to requested replay position.
- **AT-023 Exact frame-step:** Verify one actual decoded frame forward/backward per action on every supported media/browser combination. If not achieved, requirement remains failed/open; do not relabel approximate seeking as exact.
- **AT-092 Independent camera frame-step:** Select Camera 1 and verify exact one-decoded-frame stepping in both directions; repeat independently for Camera 2. Do not require both cameras to show identical frames or advance in lockstep.
- **AT-024 Zoom focal point:** Select a non-central area; zoom/pan/reset while paused; verify the selected area remains focal.
- **AT-025 No false no-signal:** Keep valid live feed while replay loads/fails; verify live picture/status is not falsely covered.
- **AT-026 Return-to-live:** Verify closing replay shows current live image, not a stale frame.

### Recording, archive and bookmarks
- **AT-030 Continuous retrieval:** Record without bookmarks; later retrieve arbitrary time intervals from both cameras.
- **AT-031 Finalization:** Normal shutdown; finalized segments are indexed and playable.
- **AT-032 Interrupted shutdown:** Use disposable test data; verify completed/incomplete segments are correctly classified after restart.
- **AT-033 Bookmark from replay:** Create a marker while replay is paused/playing; marker resolves to the exact media position viewed, not click wall-clock time.
- **AT-091 Dual-camera bookmark mapping:** Create a bookmark for a shared event; verify it stores the common event/reference time and both camera-specific media mappings, and opening it presents each available camera view with honest offset/uncertainty.
- **AT-034 Bookmark persistence:** Marker survives refresh, browser restart and app restart and leads to corresponding continuous archive.
- **AT-035 Bookmark UI:** Create marker without opening drawer; compact control remains accessible and confirmation is truthful.
- **AT-036 Live bookmark mapping:** If live-time bookmarking is supported, verify marker maps to the media time represented by the selected live delay.
- **AT-037 Bookmark retry/concurrency:** Duplicate/retried requests and two clients do not create ambiguous or lost markers.

### Storage and resilience
- **AT-040 Low storage:** Use a disposable volume; warning appears at defined, owner-approved threshold.
- **AT-041 Disk full/write stall:** Verify no automatic deletion, no false recording-healthy status, safe stop/alert and preservation of finalized data.
- **AT-042 Destination removed:** Remove disposable storage; app detects failure and does not silently switch or delete.
- **AT-043 Read/write contention:** Sustain two-camera recording while replay reads older media; verify bounded contention and no recording interruption.
- **AT-044 Explicit deletion:** Verify exact scope is displayed and explicit approval is required before any recording is deleted.

### Synchronization
- **AT-050 Two-camera sync:** Use a repeatable shared visible timing event; measure offset at start and after sustained run.
- **AT-051 Sync uncertainty:** Compare UI state/uncertainty with measured offset and drift; no unsupported accuracy claim.

### Security, platform and free interoperability
- **AT-060 Local offline core:** After installation, disconnect internet; local ingest, recording, archive, replay and bookmarks remain functional.
- **AT-061 No paid dependency:** Verify no paid account, subscription, cloud relay, paid plugin or premium external-tool feature is needed for core use.
- **AT-062 PRISM free ingest:** Test mobile/camera PRISM stream using free supported capability on a platform PRISM supports; test loss/reconnect and two-stream case where source setup allows.
- **AT-063 OBS interoperability:** Test OBS as a free source on supported Windows and Linux host platforms.
- **AT-064 Endpoint security:** Verify only documented endpoints are exposed; management/diagnostics and archive paths reject unauthorized/invalid access according to approved security policy.
- **AT-065 License compliance:** Inventory bundled dependencies and verify license/redistribution obligations before release.
- **AT-070 Clean Windows install:** Install/launch/use/update/uninstall on a clean supported Windows environment without manual config editing.
- **AT-071 Clean Linux install:** Same for each supported Linux distribution/release.
- **AT-072 ChromeOS feasibility:** Exploratory only; no release-support claim until validated.

### Health, logs and timing evidence
- **AT-080 Health independence:** Independently simulate connected/no frames, frames/no presentation, recording/no writes, archive unavailable and replay decode failure; each health state reports its own evidence.
- **AT-081 Camera identity:** Network address change/reconnect does not silently reassign archive identity.
- **AT-082 Log privacy/rotation:** Logs rotate/bound growth and diagnostic export excludes secrets and media by default.
- **AT-083 End-to-end replay trace:** Record user click, requested logical interval, archive availability, request/media span, decoded frame and first visibly presented requested frame.
- **AT-084 Visible presentation evidence:** Distinguish decoded from actually presented frame; use browser/OS instrumentation or repeatable visual evidence appropriate to the implementation.
- **AT-085 Recovery matrix:** Exercise app crash/restart, PC reboot, network/router loss, camera loss and storage loss; verify documented recovery and honest states.

### Test evidence record

Each run records test ID, requirement IDs, build/version/hash, date/time/timezone, operator, PC hardware, OS, browser, camera models/apps/settings, network topology/conditions, storage/filesystem/free space, exact steps, expected/observed result, measurement method/raw values/uncertainty, relevant logs/screenshots/traces and Pass/Fail/Blocked/Not Run status. Destructive storage tests use disposable volumes only.

Tests requiring a reference rig or numeric thresholds may initially produce exploratory measurements, not formal pass/fail against invented criteria. No Rev.8 test result counts as a Rev.9 pass.

## 16. Gate register and remaining open decisions

### Resolved by owner — must not be reopened as if undecided
1. Replay is an overlay on live; independent from live and recording.
2. Replay look-back choices 5/10/15/20/30 seconds; 15 seconds default; interval ends at Replay press.
3. Exact one-decoded-frame forward/backward stepping is required independently in either selected camera; the cameras need not step in lockstep.
4. Slow-motion Instant Replay is required; exact speed choices/audio behavior remain open.
5. Replay opens immediately at the live edge; partial availability is shown honestly rather than waiting for segment finalization.
6. One bookmark represents a shared event and stores per-camera media mappings; both available views are shown when opened.
7. Live Delay choices are 1/2/3/4/5 seconds, default 2 seconds, independent of Replay look-back.
8. No automatic deletion; controlled recording stop before storage exhaustion uses a test-derived safety reserve; explicit approval is required for any deletion.
9. Bookmarks are markers into continuous recording, not clips; no fixed pre/post-roll; bookmark from Instant Replay maps to viewed media timestamp.
10. Two cameras per mat, opposite corners, each on a tripod; automatic alignment reports measured offset/uncertainty without an unvalidated numeric guarantee.
11. Local-first access, optional password protection desired, with ease of use; Windows and Linux easy installation; ChromeOS conditional only.
12. Free-tool compatibility, no mandatory paid dependencies, local offline core after installation.
13. Supervisor feature is nice-to-have only and may be omitted/deferred.
14. Performance numbers are to be established after practical testing on a defined reference setup.

### Gate A — clarify before final implementation baseline, without reopening resolved owner decisions
- Exact local network exposure and optional password/authentication defaults.
- Minimum supported Windows/Linux releases, browser and CPU architecture.
- Initial camera/device/protocol compatibility matrix.
- Whether live-time bookmark is included in first release (preserve if low-complexity; owner has not withdrawn it).
- Exact frame-step feasibility on selected supported media/browser matrix; if infeasible, return to owner rather than silently downgrade.
- Whether a minimal multi-tournament/session model is needed for first release; supervisor/multi-mat management is not required.
- Any numeric performance targets intended as binding acceptance criteria.

### Gate B — decide after prototype/reference environment or before release as appropriate
- Reference PC/camera/router/storage/display profile and supported browser versions.
- Numeric thresholds for replay open, selected live delay, return-to-live, seek, sync, storage warnings and recovery.
- Detailed camera mounting/framing guide.
- Log retention/export, bookmark export/import and backup policy.
- Detailed accessibility/touch/display targets.
- Installer/update/rollback technology and support matrix.
- Full dependency/license inventory and release review.
- Optional supervisor/multi-mat design only if it can be added without material complexity.

## 17. Non-goals and explicit exclusions for initial core

- No mandatory supervisor dashboard or cross-mat switching.
- No automatic deletion or oldest-recording cleanup.
- No generated fixed-length bookmark clips or automatic bookmark pre/post-roll.
- No required cloud account, cloud relay, subscription, paid plugin or premium streaming-tool tier.
- No requirement to run PRISM on the PC/Linux host.
- No promise of ChromeOS support before feasibility testing.
- No requirement to preserve Rev.8 implementation choices or repair Rev.8.

## 18. Draft 0.4 change summary

Compared with Draft 0.3, this revision:
1. Carries forward the confirmed replay look-back choices/default and precise user-visible interval semantics.
2. Defines immediate replay at the live edge, without waiting for segment finalization, and truthful display of partial availability.
3. Clarifies exact independent frame stepping in either selected camera; frame-lock between cameras is not required.
4. Adds slow-motion playback as a confirmed requirement while leaving exact speed choices/audio behavior open.
5. Defines shared-event bookmarks with a per-camera mapped media position and display of both available views.
6. Confirms Live Delay choices of 1–5 seconds, default 2 seconds, independent of Replay look-back.
7. Defines controlled stop before storage exhaustion using a test-derived safety reserve, preserving recordings and prohibiting automatic deletion.
8. Carries forward marker-only bookmarks and withdraws fixed pre-roll/post-roll.
9. Retains two simultaneous cameras per mat, opposite-corner tripods, optional supervisor scope, Windows/Linux priority and conditional ChromeOS.
10. Strengthens free PRISM/OBS interoperability and offline/no-paid-dependency requirements.
11. Adds acceptance tests for live-edge replay, slow motion, independent per-camera frame stepping, dual-camera bookmark mapping, Live Delay values/alignment and controlled storage stop.
12. Keeps architecture choices and unapproved numeric performance thresholds non-binding.

**Draft 0.4 status: NOT APPROVED. This is the corrected final owner-approval candidate after the single independent Gemini + DeepSeek review round of Draft 0.3. The owner must explicitly approve this numbered specification before any implementation begins.**
