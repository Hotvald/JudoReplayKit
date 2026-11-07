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

## Test discipline

- Do not repeat these checks without a concrete reason, such as a relevant code/configuration change or a newly observed regression.
- A passing endpoint test does not prove replay playback, pause, frame stepping, or HLS segment delivery works end-to-end.
- Record exact test conditions and distinguish endpoint availability from user-visible functionality.
