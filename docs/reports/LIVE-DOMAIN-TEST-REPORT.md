# Published CyberShield test report

## Resolution

All seven defects below have now been fixed and deployed in commit `38cc213`. The 22 automated tests and 17 targeted live regression/smoke checks pass. The corrected PDF was rendered and visually verified. See [FIX-DEPLOYMENT-REPORT.md](FIX-DEPLOYMENT-REPORT.md) for deployment links and the post-fix evidence.

## Pre-fix baseline

Tested on 13 October 2026 against https://laxmiyadav72335.github.io/cybershield/ and its real backend https://cybershield-api-r4jc.onrender.com. Tests ran in Google Chrome against the uploaded version. **74 checks passed, 7 failed, 0 remained blocked.** The seven failures below are confirmed product defects; interrupted or mistimed test attempts were replaced with completed reruns and excluded from the final count.

The backend connection worked during this initial test. These were the failures before the subsequent fix/deployment request.

## Confirmed defects before the fixes

| Priority | Reproduction | Actual behavior | Fix status |
| --- | --- | --- | --- |
| High | QR Scanner → Paste QR text → `https://example.com/login?pa=merchant@paytm` → Check QR. | A web link is classified as UPI, bypassing the backend link checks. | Already fixed locally in `client/src/lib/qr.js`; not uploaded. |
| High | Submit `0:0:0:0:0:0:0:1` to the live IP API. | Expanded loopback IPv6 is marked `isPrivate: false` and receives a Safe score of 10, even though AbuseIPDB reports `isPublic: false`. | Already fixed locally in `server/services/ipScanner.js`; not uploaded. |
| Medium | Scan the known fingerprint of `hello\n` as `first-name.txt`, then scan the same fingerprint as `second-name.txt`. | The second cached report still says `first-name.txt`. | Already fixed locally in `server/services/fileScanner.js`; not uploaded. |
| Medium | Start a local UPI QR scan and use browser Back while its radar delay is running. | The cancelled scan still saves its report and navigates to the result page. | Already fixed locally in `client/src/pages/Scanner.jsx`; not uploaded. |
| Medium | Link Scanner → enter `https://[2606:4700:4700::1111]/` → Scan link. | A valid IPv6 URL is rejected by the form. | Already fixed locally in the URL validation/form code; not uploaded. |
| Medium | Focus “Skip to content” with the keyboard on a scanner page and press Enter. | The route changes to `#main` and displays 404. | Already fixed locally in `client/src/components/layout/Layout.jsx`; not uploaded. |
| Medium | Scan the supplied UPI refund-trap example and download its PDF. | The summary and score breakdown render `₹4,999` as `¹4,999`. The standard PDF font does not support the rupee glyph. | Newly confirmed; PDF font/text handling still needs correction. |

## What passed

- All 11 tested route/refresh cases: Home, scanner redirect, all six scanner forms, Result, Awareness, and the missing-page screen. A new browser session correctly shows “No scan result yet.”
- Input validation for malformed links/IPs, short messages, empty/oversized files, non-image QR uploads, and images without QR codes.
- QR decoding from an actual generated image and from synthetic camera footage. UPI refund-trap detection returned Dangerous (70/100); ordinary shop, Wi-Fi, and plain-text QR reports rendered. Ordinary QR web links reached the real backend.
- The real HIBP lookup for the synthetic test password `password` returned 52,372,427 occurrences and Dangerous (100/100). The network request sent only hash prefix `5BAA6`, with no request body. The saved report contains no raw password input.
- Public IP `8.8.8.8` returned real AbuseIPDB and geolocation results. Standard private IP `127.0.0.1` remained unscored. The expanded-spelling exception is listed above.
- A normal Google URL returned Safe (0/100), with Safe Browsing, VirusTotal, and RDAP available. Google's official phishing test URL was flagged by Safe Browsing and returned Dangerous (95/100).
- The synthetic scam SMS returned Dangerous (100/100), with Gemini and local-rule findings and visible phrase highlighting. A normal meeting message returned Safe (0/100). Some short explanations used the fallback template when Gemini did not answer within the service cap; message AI analysis succeeded.
- A generated harmless text file followed the real hash-first lookup, multipart upload, and queued-analysis flow. A separate generated file then completed the pending-analysis cycle: **Check again** used JSON hash lookup, uploaded no bytes again, and replaced the unscored report with a completed Safe result (0/100).
- Report persistence across refresh and both QR/password PDF downloads. The password PDF's visual layout passed; the QR PDF's rupee glyph defect is listed above.
- All seven Awareness cards expand/collapse. All ten quiz questions, perfect score, saved best score, and restart work.
- All 24 layout checks passed at 320, 375, and 768 px across Home, the six scanners, and Awareness. No document-level horizontal overflow was observed. The mobile navigation opens, navigates, and closes correctly.
- The public counter displays live data. One completed real test scan incremented Firestore from **19 to 20**. Writes were blocked during bulk tests to avoid inflating the public counter.
- Live API validation returned the expected 400 responses for bad URL/hash/IP and overlong message inputs, and 404 for an unknown endpoint.
- The UI displays useful network-failure and rate-limit messages. These two error cases were deliberately simulated in the test browser; the live backend's quota was not deliberately exhausted.
- No uncaught page errors or failed same-site asset responses were observed in the completed suite.

## Evidence and test limits

The detailed results are in [results.json](test-artifacts/live-domain-2026-10-13/results.json). Screenshots and downloaded PDFs are in `test-artifacts/live-domain-2026-10-13/`, including the QR dispatch failure, rejected IPv6 URL, skip-link failure, and rendered PDF pages.

Camera testing used a real browser media stream backed by generated QR footage. The user's physical webcam, camera hardware quality, and operating-system permission prompts were not tested. Only synthetic passwords/messages/files and an official phishing test URL were used. No real personal files or passwords were uploaded; the two harmless generated file samples were submitted to VirusTotal as part of the live upload tests.

The free Render service can sleep after inactivity, so a cold first request may be slow. That timing was not comprehensively measured. These checks cover the listed cases and do not establish correctness for every possible input or device.

## Original implementation plan (completed)

Publish the six existing local fixes that correspond to the confirmed live failures, correct the PDF's unsupported-glyph handling, then redeploy both the API and frontend and rerun the failing checks. The Render service was created from a public repository URL, so verify its deployment trigger or deploy the latest commit manually when publishing backend fixes. The local-only fixes are documented further in `PROJECT-REVIEW.md`.
