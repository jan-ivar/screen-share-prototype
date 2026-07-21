# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A clickable UI prototype exploring screen-share picker UX for a Firefox-in-Google-Meet
scenario. It's a single self-contained `index.html` — all HTML, CSS, and JS inline, no
build step, no dependencies, no tests. Open the file in a browser to run it.

Pushing to `main` auto-deploys the whole repo root to GitHub Pages
(`.github/workflows/deploy.yml`).

## Architecture

Everything lives in `index.html`:
- `<style>` — all CSS, organized by component with `── section ──` comment banners.
- markup — a fake Google Meet shell (`#meet`) plus one hidden overlay per flow.
- `<script>` — data tables, shared state, and the flow logic.

The prototype simulates **6 competing "flows"** (share-picker designs) selected by the
top flow bar. `currentFlow` (1–6) drives everything. `setFlow(n)` switches flows and
resets state; `onShareClick()` (the green present button) branches on `currentFlow` to
open that flow's picker.

The flow-bar button order matches `currentFlow` 1:1, and each button is labeled by its
approach: "Chrome" (the Chrome-style reference picker), then "2-way fork", "3-way fork",
"Tab-centric", "Tab grid", "SC tabs". A final button, "Navigate-to-share", is a link to
`index2.html`, not a `currentFlow`. When editing labels or adding a flow, keep the
`onShareClick()` branch and the button order in sync.

### Pickers (overlays), keyed by flow
- Flow 1 (Chrome) — `#ov-1`, full Chrome-style picker (Tab / Window / Screen tabs),
  body swapped by `renderCP1Body(type)`; `openFullPicker()` opens it.
- Flow 2 (2-way fork) — `#ov-2`, upfront 2-way fork (Tab vs Window/Screen).
- Flow 3 (3-way fork) — `#ov-3cards`, upfront 3-way fork (Tab / Window / Screen).
- Flow 4 (Tab-centric) — `#ov-4`, tab-centric picker with an "or" row of window/screen
  buttons; `openOpt4Picker()` opens it.
- Flow 5 (Tab grid) — `#ov-tabgrid`, thumbnail grid of tabs; `openTabGrid()` opens it.
- Flow 6 (SC tabs) — `#ov-5fork`, 3-way fork. Its window/screen cards call
  `showSCPicker(mode)`; its tab card calls `showTabSCPicker('content')`, opening
  `#sc-tabs-picker` in `content-mode` — a macOS-SCContentSharingPicker-style spatial view
  of windows-with-tabs where clicking a tab (`switchContentTab`) switches what that window
  previews (`renderStubContent`) and reveals a per-window "Share This Tab" overlay.

### Shared pickers reused across flows
- `#sc-picker` — the macOS window/screen picker, opened via `showSCPicker(mode)` from the
  window/screen paths of flows 3, 4, 5, and 6.
- `#sc-tabs-picker` — the SC-style windows-with-tabs view; currently only flow 6 opens it,
  via `showTabSCPicker(mode)`. The `content-mode` class toggles clickable tabs vs. always
  showing each tab's own "Share This Tab" button.
- `#ov-tabs` — a standalone tab-only picker; legacy, not wired to any current flow.

### Shared sharing model
After any picker confirms, the app enters a sharing state via one of:
- `onShareTab()` — tab sharing. Shows the simulated browser tab bar (`#sim-tabs`) and the
  `#notify-bar`, whose text changes with `activeTabView` (`meeting`/`shared`/`other`) via
  `updateNotifyBar()`. `onShareTabById(id)` / `onShareTabFromOverlay(btn)` set the selected
  tab first.
- `startWindowShare(type, label)` — window/screen sharing.

Both render a preview into `#sharing-overlay` (inside the main tile) via
`getSharingContent(type, label)` + `showSharingOverlay()`. `stopSharing()` tears down all
sharing UI and resets simulated tab labels. State globals: `sharingMode`, `activeTabView`,
`selectedTab1`, `selectedTab1Type`, `windowSharingLabel`.

### Data tables (top of `<script>`)
`TABS` (the 5 demo tabs), `WIN_DATA`, `SCR_DATA`, `TABS_MANY` (20 tabs for the
squished-tab-bar mock), `PREVIEWS` (inline-HTML tab previews), `TAB_BG` (per-tab background
colors). Tab/window content is faked HTML — there's no real media capture.

## Conventions

- Vanilla everything: no framework, no `package.json`. Event handlers are inline `onclick=`
  attributes calling global functions.
- Picker DOM is largely built by JS template-string helpers (`buildTabPickerHTML`,
  `thumbHTML`, and the local `htab`/`vtab` closures in `showTabSCPicker`), which take
  element-id arguments so one helper serves several flows.
- Visuals imitate real Chrome, Firefox, and macOS chrome closely — preserve the styling
  fidelity when editing.
