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

## Test discipline

- Do not repeat these checks without a concrete reason, such as a relevant code/configuration change or a newly observed regression.
- A passing endpoint test does not prove replay playback, pause, frame stepping, or HLS segment delivery works end-to-end.
- Record exact test conditions and distinguish endpoint availability from user-visible functionality.
