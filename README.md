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

`async-practice-walkthrough-mobile.html` is a separate file and a complete flow for
Chrome on Android and Safari on iPhone: onboarding, a live assessment with sample
questions, and the wrap-up. It is not a port of the desktop tour.

    loading -> allow access -> [the browser's own dialogs] -> camera and mic test
    -> practice question -> practice recording -> practice review -> countdown
    -> question 1 (multiple choice) -> question 2 (multimedia) -> question 3 (video
    answer, with preparation time) -> question 4 -> finish -> saving -> result
    -> feedback -> thank you

plus the states a phone produces: blocked camera, an interrupted recording, a weak
or lost connection, a save that has not landed, and a reload.

The structure is the approved wireframe. The V8 Test Taker Experience mobile frames
and the Proctoring Attendee frames are the style and layout reference. It is built
on the real Tessera components (`window.TesseraDS`); only the simulated browser
chrome, the browser's own permission dialogs and the sample camera picture are drawn
by hand.

### The frame

Every screen is the Tessera top bar, a scrolling body, and a **fixed footer that holds
every call to action at full width**. The two camera screens (the mic test and
recording) go **full page**: the camera runs behind a translucent top bar and the
footer, with the question or the device card floating over it.

### Decisions

- **Same order as web.** A candidate practices in the sequence they will meet for the
  scored questions, so the practice never teaches a different order.
- **The question comes first, then the camera.** Start recording opens the full-page
  camera and recording begins; Stop recording sits where Start recording was.
- **Preparation time lives on the question screen** of a video question in the real
  assessment, and the system starts recording when it runs out. The onboarding has no
  prep time. Three treatments can be compared from the prototype panel: the button
  fills as time runs out (the default, in a darker blue with the seconds in its
  label), a ring, or a bar above the button.
- **Stop recording saves the answer.** There is no Submit and no upload step. In
  onboarding the review says that is how it works, then offers Practice again or Start
  assessment. In the assessment it is Retake and Next. The primary action is held
  until the save has landed.
- **A multimedia question** has a clip with controls, a caption and a transcript, then
  the answer options.
- **Network messages are broadcast banners** under the top bar: weak, offline, back
  online, and the welcome-back note after a reload. After a long loss of connection the
  page is replaced by an error screen laid out like the Proctoring Attendee one.
- **A reload keeps what was saved.** Onboarding stays done, answered questions stay
  answered and the timer carries on. The question being answered when the page reloaded
  is lost and starts again, including its preparation time. A written answer saves when
  the candidate taps Next, and a video answer when the recording stops. Progress is kept
  in `sessionStorage` where the viewer allows it. After a reload the browser asks for
  the camera again, as a new page load would.
- **Blocked states** follow the Proctoring Attendee screen: a picture of a phone with
  the dialog, a plain explanation, Retry, "Try with another device", and the how-to
  link pinned at the foot. There are three: the site is blocked, Chrome itself lacks
  permission, and the camera is in use.
- **Alvy opens as a full-page overlay** under the top bar, closed with the X, so the
  timer and the bar stay visible.
- **No in-app browser screen.** It was removed by decision. Detecting a webview from the
  user agent is a guess, and the flow does not depend on it.
- **The clip plays back afterwards.** The device check only proves the live preview
  works; playback proves the recording did. Recorded with MediaRecorder, falling back
  to the last frame where that API is unavailable.

The wrap-up screens come from the Test Taker Experience file: saving, the result with a
certificate when the passing score is reached, feedback, and thank you. The result is
adapted to this sample: it grades the written questions and marks the video answer as in
review.

Opening Read more or hiding the preview mid-recording leaves the media alone. The video
and canvas are children of components that stay mounted across repaints, so a repaint
cannot replace the canvas the recorder is reading, which used to destroy the take silently.

### Permission dialogs

Drawn the way each platform really draws them, because this is the single biggest
drop-off point in the flow.

- **Android** asks twice. Chrome's own modal handles the site, then Android's runtime
  dialogs handle Chrome itself, camera and microphone separately. The OS ask only
  happens once the site is allowed.
- **iOS** shows one Safari alert. Safari asks again on every page load unless the
  candidate sets it in Website Settings.

### What the analytics changed

Three readings from the product funnel drove this work. The figures are internal: they
live on the analytics board linked from UX-1224, not in this public repo.

1. **Mobile is roughly half of candidate traffic**, and iPhone is the large majority of
   it. This is the main surface, not a slice.
2. **iPhone finishes at well under half the desktop rate**, and loses people twice: at
   the camera permission ask, and again between allowing the camera and submitting.
3. **Most of the drop at the permission step produces no event at all.** Explicit
   refusals are a small minority of it, and the instrumentation tracks only successes and
   refusals. So the largest leak in the funnel is invisible, and it is a failure, not a
   persuasion problem.

That is why the blocked, interrupted, offline, error and reload states are designed in
full rather than left as edge cases.

### Open questions and known gaps

- **Instrumentation gap.** Nothing today distinguishes "dialog never shown" from "device
  error" from "abandoned". Until it does, the drop at the permission step can only be
  guessed at. This is a product ask, not a design one.
- **Real values.** The preparation time (15 seconds here, 5 to 30 in the panel), the
  answer limit (2 minutes), the assessment length (20 minutes), the passing score and the
  point at which a lost connection becomes the error screen (15 seconds here) are
  placeholders to confirm.
- **Retention contract.** The interrupted screen says the partial answer is kept. Confirm
  that is what the backend does before the copy ships.
- **Written answers cannot be skipped.** Next stays locked until one is chosen. The
  Figma frames also show a skip-enabled variant.
- **The microphone gate still has no bypass.** Deliberate: we genuinely hear nothing, so
  the gate's judgment is certain. The counter-argument on a phone is that a silent device
  failure traps the candidate. If that is a real risk, the shape to add after a second
  failure is "Continue anyway, and we'll note we couldn't test your mic."
- **Touch targets.** Tessera's Button is 36px tall. Full-width calls to action make them
  easy to hit, and 36px passes WCAG 2.2 AA, but it is under the 44pt iOS and 48dp Android
  guidance. Worth raising with the design system team.
- **The red dot on the camera test** follows the wireframe, but reads as "recording". If
  candidates mistake it for that, label it.
- **Rating buttons and the certificate** are not Tessera components. They are built from
  tokens and flagged as new patterns.
- **The illustrations** (saving, feedback, thank you) come from Tessera's library, which
  has licence flags to clear before they leave the product.
- **The real camera path is untested here.** The recorder, the level meter and the
  interruption listeners were exercised against the drawn sample preview, because the test
  machine has no camera. Run it once on a real phone.

## Running it

No build step. Open `async-practice-walkthrough.html` in a browser, or use the
live link above. The mobile file also needs the Tessera runtime beside it, in
a `tessera/` folder (tokens, bundle and font). It is not committed to this
public repo; add it with the design-system skill's scaffold script, which
leaves `index.html` alone:

    python3 ~/.claude/skills/tessera-design-system/scripts/scaffold.py . --assets logos illustrations

Opened from disk it uses the real camera and microphone. Where those are
blocked (for example inside an embedded viewer) it falls back to a drawn
sample preview so the flow still demonstrates.

The frame is fixed at 1440×960 and scaled to fit the window. Prototype
controls sit in the panel on the right — jump to any step, switch
variant, or force a blocked state.

The mobile file scales a 360×800 (Android) or 390×844 (iPhone) frame the same
way, and draws the browser's own chrome because the recovery screens point at
it. Its panel can switch device, force any permission outcome, open the flow
as if from inside another app, drop the connection, make the save slow or
fail, and interrupt a recording in progress.
