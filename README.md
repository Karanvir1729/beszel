# Beszel: enabling two alerts in quick succession saves only the second one

- App: henrygd/beszel v0.21.1, release binaries `beszel_darwin_arm64` (hub) and `beszel-agent_darwin_arm64`, both on one machine, one system added.
- Browser: Chromium 153.0.8010.12 driven by Playwright 1.63.0, viewport 1280x720, nothing slowed (the bug is in the page's own debounce, not the connection; it also reproduced 5 of 5 on Chrome DevTools' Slow 4G preset).
- Condition: the two switches are clicked 300 ms apart.

Steps shown in the video (`beszel-alerts-two-switches.mp4`, `beszel-alerts-two-switches.gif`):

1. Dashboard, click the bell icon on the system row to open its alerts sheet.
2. Click "CPU Usage", then "Memory Usage" 300 ms later. Both switches show on.
3. Reload the page and open the sheet again: only "Memory Usage" is on. The server has one alert (Memory); only one `POST /api/beszel/user-alerts` was sent.

Counts (Playwright, server checked through `GET /api/collections/alerts/records` after each trial): 300 ms gap: 10 of 10 trials saved only Memory; 1000 ms gap (control): 0 of 10 lost an alert; 300 ms gap on Slow 4G: 5 of 5 lost CPU.

Cause: `upsertAlerts` and `deleteAlerts` in `internal/site/src/components/alerts/alerts-sheet.tsx` are one shared `debounce(..., 400)` each, and `debounce` keeps only the last call.

The black bar at the bottom of the recording (current path and the label "Overlay added by the recording script, not part of Beszel") was added by the recording script.

Screenshots: `alerts-two-switches-on.png` (both switches on after the clicks), `alerts-after-reload.png` (after reload: only Memory on).

Recorded by Claude (Claude Code) with Playwright; no one reproduced it by hand.
