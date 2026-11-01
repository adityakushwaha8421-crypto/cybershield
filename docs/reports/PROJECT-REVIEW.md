# CyberShield project review

Reviewed on 13 October 2026. Covered the React client, Node API, six scanner modules, shared scoring and cache logic, Firebase rules, dependency manifests, and GitHub Pages/Render configuration. The fixes were subsequently published and deployed in commit `38cc213`; see `FIX-DEPLOYMENT-REPORT.md`. The validation details below describe the initial review.

## Confirmed issues fixed

| Issue and reproduction | Fix | Verification |
| --- | --- | --- |
| Pasting `https://example.com/login?pa=merchant@paytm` as QR text classified it as UPI and skipped URL scanning. | Only supported payment schemes enter the UPI parser. HTTP(S) links retain their query parameters and go through the link scanner. | Browser reproduction before and after; new QR regression tests. |
| Starting a local QR scan, then using browser Back during the radar delay, still navigated to `/result` and saved the cancelled report. | The scanner checks its own abort signal after the delay, before storage, counter updates, or navigation. | Browser regression verifies the previous route and stored result remain unchanged. |
| Expanded loopback IPv6, zero-padded documentation IPv6, and hexadecimal IPv4-mapped loopback addresses bypassed non-public checks. | Canonicalize IPv6 and convert IPv4-mapped addresses before classification, lookup, and caching. | Unit tests verify these addresses are non-public and remain unscored without external lookups. |
| VirusTotal statistics containing only zeroes, unfinished results, or only failed/timed-out checks were treated as completed detection data. | Require a finished analysis and at least one actual detection/classification result before exposing statistics. | Regression tests verify these responses remain queued with no scoreable statistics. |
| Scanning identical file contents under another filename returned the previous file's name and MIME type from cache. | Reuse cached detection findings while preserving the current request's file metadata. | Regression test verifies current name, preview, MIME type, and cached score. |
| The link form rejected valid IPv6 URL hosts supported by the backend. | Align client validation with supported HTTP(S) domain/IP hosts and the API's length limit. | New validation tests plus a browser submission using an IPv6 URL. |
| Keyboard activation of “Skip to content” in the GitHub Pages build changed the hash to `#main` and opened a 404 page. | Focus and scroll to the main content without changing the route; make the main element focusable. | Production browser reproduction before and after; route stays `#/scan/url` and focus reaches `main`. |
| Dependency audit reported four high-severity package entries originating from Firebase's outdated `@grpc/grpc-js` dependency, version 1.9.16. | Pin the transitive dependency to patched version 1.14.6 with an npm override and update the client lockfile. | Final full audits of root, server, and client all report zero known vulnerabilities. The implicated package is the Node gRPC transport; the audit findings do not establish exploitation of this browser app. |

Also corrected the advertised Node version to match the installed tools, the server's engine requirement to match its Gemini SDK, and stale Vercel documentation to match the existing GitHub Pages deployment workflow.

## Checks completed

- Automated tests: **21 passed, 0 failed** (15 server, 6 client), up from 14 baseline tests. New regressions failed against the original affected logic before fixes where applicable.
- Client lint passes. Git whitespace checks pass.
- Standard production build and GitHub Pages build pass. The build still warns about the main JavaScript chunk, approximately 894 kB before compression / 278 kB compressed.
- Browser checks cover all ten main/empty/error routes, all six scanner submission flows, invalid URL/IP inputs, empty-file rejection, file hashing and hash-first upload flow, pending file results, private-IP results, message highlighting, password breach handling, report refresh persistence, and PDF download.
- QR link dispatch, password breach responses, file upload responses, and the scam-message UI response were simulated in browser regression checks. Password testing used the synthetic value `password`; only its five-character hash prefix was requested and the saved report contains no raw password field.
- All ten quiz answers, score persistence, and restart pass.
- Mobile checks at 375 px show no document-level horizontal overflow on Home, Link Scanner, QR Scanner, and Awareness. Browser tests report no uncaught page errors.
- Production GitHub Pages checks verify eight routes and reloads under `/cybershield/`, including the fixed keyboard shortcut.
- Live API checks verify expected 400/403/404 responses for malformed inputs, overlong messages, invalid hashes, blocked origins, and missing routes. The allowed local origin succeeds.
- Three synthetic live scans succeeded: Google URL lookup returned completed Safe Browsing, VirusTotal, RDAP, and Gemini results; `8.8.8.8` returned AbuseIPDB, geolocation, and Gemini results; a normal meeting message returned Gemini and local-rule results.

## Limits and remaining work

- Physical camera decoding, real QR image decoding, live unknown-file upload to VirusTotal, actual HIBP breach data, and live Firestore counter writes were not verified. Firestore traffic was blocked during browser testing to avoid changing the public counter. The PDF download was verified; its full visual layout and non-Latin text were not assessed.
- The live API was tested as already running. The changed backend edge cases are verified by tests importing the updated source; restart the API to pick up those changes.
- The initial development launch hit sandbox file-watch and listening restrictions. Testing continued with the normal API process and an authorized local web server. These errors are environment limitations, not counted as project bugs.
- The initial production preview used the wrong base-path setting; it was corrected to match the GitHub Pages build before production validation.
- The large main bundle remains a performance improvement opportunity. No hosting deployment or live website update was performed.

## Short handoff for Claude

Review the local changes in `PROJECT-REVIEW.md`. Scanner fixes cover QR URL dispatch, cancellation during the radar delay, private IPv6 normalization, pending VirusTotal results, cached file identity, IPv6 link validation, and GitHub Pages skip-link navigation. Firebase's gRPC dependency is overridden to 1.14.6; all three dependency audits are clean. Tests: 21/21; lint and both builds pass. Review the remaining bundle-size warning and the unverified hardware/provider paths listed above. Do not assume the running API has reloaded the changed source.
