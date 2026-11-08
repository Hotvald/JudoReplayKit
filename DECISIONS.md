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


## Rev.9 requirements direction — free-tool compatibility and no paid dependencies (owner-confirmed 2026-11-08)

**Decision:** JudoReplayKit must be fully usable for its intended core purpose without mandatory paid subscriptions, paid cloud services, paid streaming platforms, paid runtime components or paid third-party licenses.

Requirements:
- Core live ingest, continuous recording, archive retrieval, replay overlay, bookmarks, two-camera operation (within the agreed supported scope), diagnostics and local operation must not require an ongoing payment.
- Do not make optimal or essential operation depend on a vendor cloud account, hosted relay, subscription API, paid codec, paid plugin, or premium tier.
- Prefer open, documented, interoperable protocols and freely available tools. Avoid binding the product to one streaming application.
- Provide and test a documented integration path for free streaming software. PRISM Live Studio is an explicitly requested compatibility target where its supported platform and free features permit; OBS Studio is a strong cross-platform compatibility target, including Linux.
- Do not require PRISM Plus or any other paid feature. If a third-party tool offers optional paid extras, JudoReplayKit's integration and core workflow must use only the free capabilities.
- Third-party components must be license-compliant and redistributable under the selected distribution model. A component that requires a paid commercial license for the intended use is not an acceptable mandatory dependency.
- Installation, updates, normal use and local replay must not require a paid account or internet connection after installation, except where the user separately chooses an external streaming destination/service.

Compatibility clarification (checked 2026-11-08):
- PRISM's official desktop information currently lists Windows and macOS downloads, not Linux. Its FAQ documents Custom RTMP and other streaming protocol support, and its announcement says core live streaming remains free while selected premium features are subscription-only. Therefore, test PRISM compatibility on a supported PRISM platform using its free Custom RTMP capability; do not promise a native PRISM Linux client.
- OBS Studio is officially presented as free/open-source software with Windows, macOS and Linux downloads. It is an appropriate cross-platform compatibility target, subject to protocol and version testing.

This is a product requirement, not approval of a particular protocol, media server, package format or architecture. The exact supported protocol matrix and compatibility versions must be specified and acceptance-tested. No paid vendor or streaming product is mandatory.


## Rev.9 owner decisions — 2026-11-08 (follow-up)

The owner has confirmed the following product requirements and preferences for the final Rev.9 specification:

1. **Replay interval:** The proposed choices (5, 10, 15, 20 and 30 seconds; 15 seconds default) are accepted on the explicit interpretation that the selected duration is the look-back interval ending at the instant Replay is pressed. Example: pressing Replay at 14:32:10 with 15 seconds selected requests the interval 14:31:55–14:32:10. This defines the user-visible interval, not the underlying API/container retrieval span.
2. **Frame stepping:** Choose option A: the product promise is one actual decoded video frame per step, forward and backward, subject to feasibility and verification in the supported playback implementation. Do not silently substitute approximate time jumps.
3. **Bookmarks during replay:** The operator must be able to create a bookmark while viewing Instant Replay. The bookmark must identify the relevant event/media time in the replayed sequence, not merely the wall-clock time when the button is pressed. Keep the proposed retrieval window of 30 seconds before and 15 seconds after the bookmarked event. The implementation must account for post-roll footage becoming available after the event.
4. **Retention/deletion:** Choose option C: no automatic retention-based or low-space deletion. The user manages recordings. Any deletion of recordings must require explicit user approval before it happens. The UI must make the selected items/scope clear and warn about low storage; do not delete automatically to recover space.
5. **Two-camera synchronization:** Choose option A: attempt automatic synchronization and report measured alignment/uncertainty honestly. Do not claim a guaranteed numeric tolerance unless later validated and explicitly approved.
6. **Network access:** Choose option C, subject to usability: local access should be straightforward by default, with the option to enable password protection. The security design must not add unnecessary friction to ordinary local operation; exact exposure defaults and authentication flow must be specified without compromising safety.
7. **Tournament/mat scope:** NOT DECIDED. The owner did not understand the distinction in the earlier question. Explain the practical difference and obtain an explicit answer before freezing scope. Do not assume single-mat or multi-mat support.
8. **PRISM usage:** PRISM is intended to run on the mobile device or camera that sends/streams video to JudoReplayKit; it is not requested as a desktop control/production dependency. Integration should therefore focus on receiving the stream from the mobile/camera source using free capabilities. Preserve protocol-level interoperability and the no-paid-dependency requirement. OBS remains a possible free PC/Linux test source/compatibility tool where relevant, not a requirement to run PRISM on Linux.
9. **Performance:** The owner agrees that immediate replay, independent live/recording operation and straightforward return-to-live are functional requirements now, while exact numerical timing thresholds are established after practical tests on a defined reference setup.

These are owner-confirmed product decisions/preferences except item 7, which remains open. They do not select media-server, protocol, database, installer or other implementation technology. Implementation remains gated on approval of the final numbered Rev.9 specification.


## Rev.9 owner clarification — bookmarks, multi-mat workflow and per-mat equipment (2026-11-08)

**Bookmarks — clarified owner intent:** A bookmark is only a marker/pointer to a moment in the continuous recording, primarily for post-tournament learning/review. It is not a request to create or preserve a fixed-length clip. The previously suggested automatic 30-second pre-roll and 15-second post-roll are NOT required and are withdrawn. The continuous recording is the source that allows the user to inspect whatever surrounding sequence is useful later. The bookmark should identify the correct media/event time and take the user directly to that point in the archive. The owner previously required bookmark creation to be possible from Instant Replay; preserve this, mapping the marker to the media position currently being viewed (not the button-click wall clock). The owner has not withdrawn the ability to bookmark during live operation. Do not make clip generation or automatic pre/post-roll a core bookmark behavior.

**Multi-mat intent:** Multiple mats are desirable if a supervisor can switch between all mats to view their live feeds and Instant Replay. However, the primary and non-negotiable workflow is that each mat referee has a straightforward local workstation with live video at a selected delay and immediate Instant Replay. Multi-mat/supervisor functionality must not degrade or complicate the mat referee workflow, and must not introduce replay latency or interrupt live display/recording. Treat independent per-mat operation as the baseline; supervisor cross-mat viewing/switching is a desired capability whose architecture and scope must be evaluated. The owner has not yet specified a maximum number of mats or whether supervisor control must be available in the first release.

**Expected equipment per mat (owner planning assumption):**
- One PC, preferably supporting Windows and Linux; ChromeOS is desirable only if it can be supported without disproportionate complexity.
- One router.
- Two mobile phones or other streaming cameras (e.g. GoPro-class devices).
- Power strip and/or extension cable reel with outlets.
- One wireless mouse.

This is the owner's anticipated deployment kit, not a finalized hardware compatibility matrix. The Rev.9 design/test plan must assess two simultaneous camera streams per mat, local network operation through the per-mat router, selected-delay live viewing, continuous recording and Instant Replay on the PC. Windows/Linux are confirmed platform priorities. ChromeOS is an exploratory/conditional target, not a promised supported platform until feasibility and browser/runtime constraints are assessed. Camera support must be based on documented/tested stream protocols and actual device compatibility, not brand assumptions.
