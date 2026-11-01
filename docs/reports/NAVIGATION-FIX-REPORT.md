# Blank scanner page: cause and fix

The screenshot showed a real frontend defect. After normal navigation from Home to Scanner, the scanner heading and form existed in the DOM, but the surrounding page wrapper stayed at `opacity: 0` and `translateY(-10px)`. No JavaScript exception was needed for the page to look blank.

The route shell used `AnimatePresence mode="wait"` around a keyed `motion.div`, while route redirects and lazy-loaded pages updated the outlet. The exiting wrapper could finish its fade-out while containing the new page and leave that content transparent. The issue reproduced with a deliberately slower scanner download and also when opening Awareness.

The earlier checks largely used route reloads and reduced motion. They missed this normal-navigation path. The new checks use normal motion, navigate through links and redirects, and verify that the heading and every ancestor are visibly rendered rather than merely present in the DOM.

## Change

The route shell now renders a normal visible container immediately, with no exiting page animation gating the next route. A `Suspense` boundary inside the page area shows a spinner during lazy loading while keeping the navigation visible. The error boundary is still keyed to the route so scanner cleanup and error recovery happen when leaving a page.

## Verification and deployment

- 22 automated tests pass; client lint and production build pass.
- 26 local navigation checks pass with normal motion: first load, slow page loader, scanner redirect, all six tabs, Home cards, Awareness, and browser Back/Forward.
- Published commit: [8ceea3b](https://github.com/laxmiyadav72335/cybershield/commit/8ceea3ba3514d2da27953c2281610ee0bdf6ae2c).
- GitHub Pages rebuild: [deployment run](https://github.com/laxmiyadav72335/cybershield/actions/runs/38405939125).
- GitHub Pages deployed the fix successfully.
- **26 live navigation checks pass, 0 fail**, with normal motion and delayed page downloads. They verify both the correct route content and the visibility of every ancestor of its heading.
- The affected Chrome tab was refreshed to the corrected scanner deployment.

Evidence is in `test-artifacts/navigation-fix-2026-10-13/`. The API code was unchanged, so no API deployment was required for this frontend correction.
