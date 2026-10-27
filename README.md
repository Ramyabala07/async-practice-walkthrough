# Async interview practice walkthrough

Clickable prototype for the Talview Tessera async interview onboarding
(UX-1224) — the first-run walkthrough a candidate sees before their first
recorded answer.

**Live prototypes**

- Desktop — https://ramyabala07.github.io/async-practice-walkthrough/
- Mobile — https://ramyabala07.github.io/async-practice-walkthrough/async-practice-walkthrough-mobile.html

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

## Mobile

`async-practice-walkthrough-mobile.html` is a separate file carrying the
same two variants and the same copy, re-laid-out for a phone:

- A phone frame — Android Chrome 360x800 or iPhone Safari 390x844 —
  switched from the prototype panel. The browser chrome is drawn because
  the blocked-permission steps point straight at it: the spotlight lands
  on Chrome's lock icon or Safari's aA button, each with its own
  recovery steps and its own native-looking permission prompt.
- The camera preview is full-bleed portrait, the way Meet, Zoom and
  Teams show a pre-call check.
- Every tour step is a bottom sheet in normal flow rather than an
  overlay, so it takes its own height out of the camera area. A step can
  never cover what it points at, and the primary action is always within
  thumb reach. Touch targets are 48px.

The preview keeps running from the device check onward, unlike the
desktop frame, where it restarts at the recording step: on a phone a
full-bleed dark rectangle reads as a fault, and both variants already
have permission by that point.

## Running it

Single HTML file, no build step. Open `async-practice-walkthrough.html`
in a browser, or use the live link above.

Opened from disk it uses the real camera and microphone. Where those are
blocked (for example inside an embedded viewer) it falls back to a drawn
sample preview so the flow still demonstrates.

The frame is fixed at 1440×960 and scaled to fit the window. Prototype
controls sit in the panel on the right — jump to any step, switch
variant, or force a blocked state.
