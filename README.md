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
same two variants and the same copy, redesigned for a phone. It is not a
port of the desktop tour: there is no spotlight, no coach mark and no
step sheet floating over the page. The onboarding is a linear run of
full screens —

    loading -> allow (priming) -> the browser's own permission dialogs
    -> device check -> question -> get ready -> record -> all set
    -> countdown

Light screens carry the instructions. The camera takes the whole screen
only where the camera is the point: the device check, getting ready, and
recording. During recording the question sits one tap away behind a
"View question" chip rather than covering the preview.

### Permission dialogs

Drawn the way each platform really draws them, because this is the
single biggest drop-off point in the flow.

- **Android** asks twice. Chrome's own modal handles the site, and then
  Android's runtime dialogs handle Chrome itself — camera and microphone
  separately, so a candidate who has never used the camera in Chrome
  sees three dialogs in a row. The OS ask only happens once the site is
  allowed. The prototype panel can start with the OS permission already
  held, which is the common case for daily phone users.
- **iOS** shows one Safari alert. Safari asks again on every page load
  unless the candidate sets it in Website Settings, so the priming
  screen says so.

Three distinct blocked screens follow from this: the site blocked the
request, Chrome lacks the OS permission, or another app holds the
camera. Each carries the steps for that phone, and the site-blocked one
draws a small mock of the address bar showing which control to tap —
Chrome's lock icon at the top, Safari's aA at the bottom.

## Running it

Single HTML file, no build step. Open `async-practice-walkthrough.html`
in a browser, or use the live link above.

Opened from disk it uses the real camera and microphone. Where those are
blocked (for example inside an embedded viewer) it falls back to a drawn
sample preview so the flow still demonstrates.

The frame is fixed at 1440×960 and scaled to fit the window. Prototype
controls sit in the panel on the right — jump to any step, switch
variant, or force a blocked state.
