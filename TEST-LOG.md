# Test Log — Rev.8

## 2026-11-06 — Verified checks

Test installation: isolated Rev.8 environment.

| Check | Result | Detail |
|---|---|---|
| Web application | PASS | Test web endpoint returned HTTP 200. |
| MediaMTX API | PASS | Test API endpoint returned HTTP 200. |
| Playback list cam1 | PASS | Endpoint returned HTTP 200. |
| Playback list cam2 | PASS | Endpoint returned HTTP 200. |
| cam1 source | PASS | Ready=True, Available=True, source=rtmpConn. |
| cam1 live picture | PASS | Live picture was visible in the browser; no “No signal” overlay during this check. |
| Continuous recording | PASS | Recording file grew from 35,424,638 to 39,837,454 bytes over 10 seconds (+4,412,816 bytes). |
| cam2 | EXPECTED OFFLINE | Phone/camera 2 was switched off during the check. |

## 2026-11-07 — Rev.8 startup and visible-picture check

**Source:** User report in the JudoReplayKit project conversation.  
**Test scope:** Isolated Rev.8 test installation; production installation excluded.

| Check | Result | Detail |
|---|---|---|
| Rev.8 start script | PASS | User started `START-REV8-TEST.bat`; browser opened. |
| Test server ports | PASS | At the time of the check, ports 1936, 8081, 8889, 8890, 9995, 9998 and 9999 were reported listening. |
| Test web endpoint | PASS | `http://localhost:8081/` returned HTTP 200. |
| Picture visible in application | PASS — user observed | User confirmed that the application was running and the picture was visible. |
| Exact mobile-stream state | NOT RECONFIRMED IN LATEST MESSAGE | Do not infer additional camera or recording state from the short user confirmation alone. |
| Recording file creation/growth | NOT YET VERIFIED IN THIS SESSION | Must be checked separately; visible picture alone does not prove recording. |
| HLS playlist and segment delivery | NOT YET VERIFIED IN THIS SESSION | End-to-end delivery through the test Caddy route remains to be checked. |
| Instant replay and replay controls | NOT YET TESTED IN THIS SESSION | Pause, frame stepping, bookmark behavior and zoom remain outstanding. |

**Important qualification:** The picture-visible result above is a direct user observation, not an independent measurement by the assistant. No claim is made here that recording, HLS segments, or replay are working.

## 2026-11-07 — Additive correction to startup record

A previous draft stated that the mobile stream had been started during this test. That statement was stronger than the latest user confirmation supported and is qualified in the project log. The original wording remains visible in Git history.

## 2026-11-07 — Recording and playback endpoint verification

**Source:** User-provided PowerShell output and direct UI observation.  
**Test installation:** `D:\JudoReplayKit_Rev8_Test`. Production installation was not modified.

| Check | Result | Detail |
|---|---|---|
| MediaMTX process | PASS | Process PID 1368 was present in the check. |
| Caddy process | PASS | Process PID 11792 was present in the check. |
| Test ports | PASS | 1936, 8081, 8889, 8890, 9995, 9998 and 9999 all returned TCP True. |
| cam1 recording file | PASS — growth observed | File `2026-11-07_12-31-46-615665.mp4.mp4` was first seen at 0 bytes, then later at 113,620,191 bytes; LastWriteTime 12:37:36. |
| cam1 path status | PASS | MediaMTX API: `ready=True`, source type `rtmpConn`; one HLS reader. |
| cam2 path status | EXPECTED NOT READY | `ready=False`, no source and no readers. |
| MediaMTX playback list | PASS | `http://127.0.0.1:9995/list?path=cam1` returned HTTP 200 and one recording, start 12:31:46.615665+01:00, duration 636.333 seconds. |
| Caddy playback list | PASS — list only | `http://127.0.0.1:8081/playback/list?path=cam1` returned HTTP 200 and the same recording entry. |
| Direct MediaMTX media GET | PASS — response headers | `/get?... ` returned HTTP 200, Content-Type `video/mp4`. |
| Caddy media GET | PASS — response headers | `/playback/get?... ` returned HTTP 200, Content-Type `video/mp4`. |
| Caddy path without prefix | FAIL — expected route mismatch | `http://127.0.0.1:8081/get?... ` returned HTTP 404. |
| MP4 data read through Caddy | PASS | 1,024 bytes read; leading bytes `00 00 00 20 66 74 79 70 6D 70 34 32` match an MP4 `ftypmp42` header. |
| Instant Replay UI | FAIL — no picture | User reports replay window opens but no video appears. After closing it, live picture is still visible. |
| Recording while replay window open | NOT INDEPENDENTLY VERIFIED | UI message says replay does not affect recording; user sees live picture again after closing replay. No separate file-growth measurement was made during the replay-window interval. |
| Duplicate file extension | FAIL | Observed filename ends in `.mp4.mp4`. |

### Interpretation and next action

The recording file contains written data, MediaMTX lists it, and both direct MediaMTX `/get` and Caddy `/playback/get` return MP4 data. Therefore, the black replay window is not explained by a missing recording alone.

A path mismatch is a concrete candidate: the Caddy playback-list response returns a media URL on `http://127.0.0.1:8081/get?... ` (without `/playback/`), while the tested working Caddy media route is `/playback/get`; the unprefixed route returns 404. This has not yet been proven to be the exact cause of the replay player's failure.

Next: inspect the exact Rev.8 replay-player code, the URL returned by the playback-list API, and the browser's media error/console behavior. Do not change production, stop running services, or assume the replay-player fix before inspecting current files.

## Test discipline

- Do not repeat these checks without a concrete reason, such as a relevant code/configuration change or a newly observed regression.
- A passing endpoint test does not prove replay playback, pause, frame stepping, or HLS segment delivery works end-to-end.
- Record exact test conditions and distinguish endpoint availability from user-visible functionality.


## 2026-11-07 — Follow-up: actual playback-list body

**Source:** User-provided PowerShell output. **Test installation:** Rev.8 only; no code/config changes made.

| Check | Result | Detail |
|---|---|---|
| Config segment duration | OBSERVED | `recordSegmentDuration: 1h`. |
| Direct MediaMTX playback-list body | PASS — non-empty | One cam1 span: start `2026-11-07T12:31:46.615665+01:00`, duration `1478.433` seconds (about 24m 38s). |
| Caddy playback-list body | PASS — non-empty | Same span returned. Generated media URL uses `/get` without `/playback/`. |
| Empty-list hypothesis | DISPROVED for this request | The actual JSON body contains a span; the player has data to consider. |
| Configured recording path | OBSERVED | Config points to `C:\\JudoReplayKit_Rev8_Test\\Videos\\...`. |
| Folder enumerated by user's command | DIFFERENT PATH | Command listed `D:\\JudoReplayKit_Rev8_Test\\Videos` and showed four Nov 6 files. This does not verify contents of the active C: recording path. |
| Replay video rendering | STILL FAILING / UNRESOLVED | Need overlay status text and browser Network/Console evidence to see whether app requests `/playback/get`, receives data, and decodes frames. |

**Next diagnostic step:** Check the configured C: recording folder, then inspect the replay overlay's exact status and DevTools Network/Console while opening Instant Replay. Do not modify files until the request/decode path is identified. The `.mp4.mp4` naming issue remains separate and unmodified.


## 2026-11-07 — Active C: recording folder check

**Source:** User-provided PowerShell output. **Test installation:** Rev.8 only.

| Check | Result | Detail |
|---|---|---|
| Active configured recording folder | PASS | User checked `C:\\JudoReplayKit_Rev8_Test\\Videos`, matching `recordPath` in the active config. |
| Current cam1 recording file | PASS — exists | `cam1\\2026-11-07_12-31-46-615665.mp4.mp4`. |
| Current cam1 recording size | PASS — substantial data | 478,547,112 bytes (about 456 MiB). |
| Recording discoverable via playback list | PASS | Earlier actual list body contained a cam1 span of 1,478.433 seconds. |
| Instant Replay video display | UNRESOLVED | Recording existence and list discovery do not establish browser decode/render success. |

**Next:** Record the exact status/wait text shown in the Instant Replay overlay after waiting about 20 seconds; then inspect browser Network/Console for the actual media request and decode errors. No repair yet. Production untouched. Duplicate `.mp4.mp4` remains separate.


## 2026-11-07 — Instant Replay overlay status: “no completed recording found”

**Source:** User's direct observation. **Test installation:** Rev.8 only. The application/replay window was already running; no restart was performed.

| Check | Result | Detail |
|---|---|---|
| Instant Replay overlay status | FAIL — exact UI message | User reports the bottom of the window says **“no completed recording found”**. |
| App/replay window state | RUNNING | User asked whether it should be closed/reopened and confirmed it was currently running. Assistant advised leaving it open. |
| Recording file existence/data | PREVIOUSLY VERIFIED | Configured C: cam1 file existed at 478,547,112 bytes (~456 MiB). |
| MediaMTX playback-list API | PREVIOUSLY VERIFIED PASS | Direct /list returned a non-empty cam1 span (duration 1,478.433 seconds). |
| UI completed-recording lookup | UNRESOLVED | Exact request, path, time window, response parsing, and “completed” criteria have not yet been inspected. |
| Production installation | UNTOUCHED | No production changes or service restarts. |

### Next diagnostic step

Find the exact source-code branch that emits “no completed recording found” in the current Rev.8 app.js/HTML. Trace the lookup request and compare it with the known successful MediaMTX /list response and Caddy /playback/list response. Check browser Network/Console only as needed to capture the actual request and response. Determine whether the UI is querying a different path, time range, or completed-recording condition. Do not make a speculative fix or restart the app/services. Do not modify production.


## 2026-11-07 — Review of proposed `openInstantReplay()` race-condition diagnosis

**Source:** Second AI's diagnosis as relayed by the user; independent review in ChatGPT. **Status: NOT VERIFIED; no repair applied.**

| Item | Assessment | Detail |
|---|---|---|
| Suspected code | HYPOTHESIS | `openInstantReplay()` filters spans with `s.end <= now + 350`, where `now` is captured before asynchronous list requests. |
| Proposed correction | NOT APPROVED YET | Recalculate `Date.now()` after `Promise.all()` and consider a larger future tolerance. |
| Evidence for stale-time race | INSUFFICIENT | Need the actual request/response timing and `s.end` value for the failing click. |
| Arithmetic cross-check | IMPORTANT | Previously logged span start `12:31:46.615665+01:00` plus duration `1478.433s` gives end about `12:56:25.048+01:00`. Compare with actual click time; this stored span alone does not show a future end. |
| Tolerance of 2,000 ms | UNJUSTIFIED SO FAR | A fresh post-fetch timestamp is reasonable, but the extra tolerance should be supported by measured timing and must not hide another issue. |
| Production / running services | UNCHANGED | No code, config, or process changes. |

**Next:** Obtain the exact current Rev.8 `openInstantReplay()` implementation and the actual `/playback/list` JSON response plus request timing for one failing attempt. Calculate `s.end` and compare against both pre-request and post-response browser timestamps. Only then decide whether a minimal code change is justified. Do not restart the app or services.


## 2026-11-07 — Browser Network check: cam1 list succeeds, cam2 list returns 404

**Source:** User-provided DevTools Network details and copied cam1 Response JSON. **Test installation:** Rev.8 only. No code/configuration or service changes.

| Check | Result | Detail |
|---|---|---|
| cam1 /playback/list | PASS | HTTP 200. |
| cam1 response body | PASS — non-empty | One item: start 2026-11-07T12:09:52.409Z, duration 301 seconds. |
| cam1 request range | OBSERVED | URL start 12:09:52.409Z, end 12:14:53.409Z UTC. |
| cam1 span end calculation | EXACT MATCH | Start + 301 seconds = 12:14:53.409Z, exactly the request's end boundary. |
| cam1 returned media URL | PATH MISMATCH CANDIDATE | Response URL is http://localhost:8081/get?... , without /playback/. Earlier direct test found Caddy /get returns 404 and /playback/get returns 200. Whether app.js uses the response URL remains unverified. |
| cam2 /playback/list | HTTP 404 | Consistent with previously observed cam2 not sending/no source; does not imply cam1 failure. |
| cam2 HLS playlist | HTTP 404 | Same offline-camera context; not evidence of cam1 failure. |
| Instant Replay status | STILL FAILING | User had reported “no completed recording found”. |

### Updated assessment of the time-filter hypothesis

The supplied current request and response make the filter issue substantially more likely than the earlier generic latency explanation:

- openInstantReplay() constructs the query end as new Date(now + 1000).
- The browser request's end is 12:14:53.409Z.
- The returned span starts at 12:09:52.409Z and has duration exactly 301 seconds, so its calculated s.end is 12:14:53.409Z.
- The supplied code accepts only s.end <= now + 350.
- If the now used to construct the request is the same captured now used by the filter, the returned span is at now + 1000 and is rejected by 650 ms.

This is a concrete boundary mismatch indicated by the actual request and body, not just a hypothetical >700 ms network delay. It strongly supports why cam1 contributes no accepted span and why the “no completed recording found” branch is reached. However, the exact current local source and runtime timestamp values have not yet been independently re-read/logged in this chat; do not silently upgrade this to absolute proof.

### Next diagnostic step

Inspect current Rev.8 source to confirm the same pre-request now is used in the filter. Also inspect whether replay media loading consumes the JSON url (which omits /playback/) or uses the app's separate playbackUrl() builder (which reportedly creates /playback/get). Do not modify code, restart services, or touch production before the user approves a specific repair plan.
