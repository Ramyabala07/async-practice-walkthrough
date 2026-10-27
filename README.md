# Async interview practice walkthrough

Clickable prototype for the Talview Tessera async interview onboarding
(UX-1224) — the first-run walkthrough a candidate sees before their first
recorded answer.

**Live prototype:** https://ramyabala07.github.io/async-practice-walkthrough/

## What it covers

A spotlight tour with anchored coach marks over the real candidate screen,
in two variants:

- **Variant A** — a separate device check screen before the practice
  question, then a 3-step tour (question → start → answer).
- **Variant B** — the same checks built into the tour as steps 1–2, so the
  5-step flow runs allow access → check devices → question → start → answer.

Both cover camera and microphone permissions, the blocked and
camera-in-use states with per-browser recovery steps, live preview and
level metering, and a Chat with Alvy panel for help.

## Running it

Single HTML file, no build step. Open `async-practice-walkthrough.html`
in a browser, or use the live link above.

Opened from disk it uses the real camera and microphone. Where those are
blocked (for example inside an embedded viewer) it falls back to a drawn
sample preview so the flow still demonstrates.

The frame is fixed at 1440×960 and scaled to fit the window. Prototype
controls sit in the panel on the right — jump to any step, switch
variant, or force a blocked state.
