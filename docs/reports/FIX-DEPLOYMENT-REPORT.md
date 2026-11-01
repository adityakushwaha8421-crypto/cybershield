# CyberShield fixes deployed

## Subsequent navigation correction

The user's blank-page screenshot exposed a further route-animation defect that the initial reload-based checks missed. It was fixed and deployed in commit `8ceea3b`. The corrected frontend passes 26 normal-navigation checks locally and 26 on the domain, including lazy loading, redirects, all scanner tabs, and Back/Forward. See `NAVIGATION-FIX-REPORT.md` for the cause and evidence. The earlier seven fixes below remain deployed; the backend is still on `38cc213` because this correction changes only the frontend.

All seven defects confirmed in the live-domain review are fixed and deployed to the frontend and API.

- Published commit: [38cc213](https://github.com/laxmiyadav72335/cybershield/commit/38cc213ffbe9cbf428eebe9bf4dafaadd1e9af39).
- GitHub Pages deployment: [successful run](https://github.com/laxmiyadav72335/cybershield/actions/runs/38401270216).
- Render deployment: [successful API deploy](https://dashboard.render.com/web/srv-db71m6ss728c73d54qj0/deploys/dep-db72rkbrjlhs739ojpcg), using the same commit.
- Live frontend: https://laxmiyadav72335.github.io/cybershield/
- Live API: https://cybershield-api-r4jc.onrender.com

## Fixes verified on the domain

1. Web QR codes containing `pa=` now enter the Link Scanner instead of being mistaken for UPI requests.
2. Leaving a scan during its radar delay cancels result storage and late navigation.
3. Valid IPv6 URL input reaches submission. This frontend dispatch test intercepted its provider request to avoid an unnecessary lookup; real backend URL checking was verified through the QR link test.
4. “Skip to content” keeps the current route and focuses the main content.
5. UPI PDFs print `INR 4,999` correctly in the summary and breakdown. The downloaded live PDF was text-checked, rendered, and visually inspected.
6. Expanded loopback IPv6, documentation IPv6, and hexadecimal IPv4-mapped loopback normalize correctly and remain unscored.
7. Cached file reports retain the current filename, preview, and MIME type.

The update also includes the earlier VirusTotal statistics fix and the patched Firebase dependency. Private scanner keys remain in Render. Publishing did not add a deployment key or change repository access controls.

## Verification

- **22 automated tests pass:** 15 server and 7 client, including the new PDF encoding regression.
- Client lint and GitHub Pages production build pass.
- **17 live regression/smoke checks pass, 0 fail:** the seven previously failing cases, eight published pages, mobile layout, and browser error monitoring.
- All 21 uploaded files were verified by their Git blob hashes against the tested local files.
- Detailed live results: [results.json](test-artifacts/post-fix-2026-10-13/results.json).
- Corrected PDF and rendered preview: `test-artifacts/post-fix-2026-10-13/upi-report.pdf` and `upi-report.png`.

The earlier 81-case report remains as the pre-fix baseline. The complete 81-case suite was not rerun after deployment; the failed cases and relevant page/layout checks were rerun. Physical webcam hardware and operating-system camera prompts remain outside testing. The production build still has its existing main-bundle size warning.

## Local repository

The published commit was fetched and the local branch was rebased onto it. All deployed functional files match the tested local files. A one-line explanatory comment in `client/.env.example` remains local because GitHub's folder upload skipped that hidden example file. Review reports and test artifacts are local and were not included in the source upload.
