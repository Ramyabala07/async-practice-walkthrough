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

`async-practice-walkthrough-mobile.html` is a separate file. It is not a port
of the desktop tour — no spotlight, no coach marks, no step sheet over the
page. It follows the web order and the approved wireframe:

    loading -> permissions -> [the browser's own permission dialogs]
    -> test device -> practice question -> record -> review -> countdown

plus the states a phone actually produces: an in-app browser, an interrupted
recording, and a save that has not landed. Those are in **What the analytics
changed** below, and they are not edge cases — they are where the funnel
leaks.

What the flow rests on:

- **Same order as web.** A candidate practises in the sequence they will meet
  for the scored questions, and there is one flow to spec and QA rather than
  two. The permission screen reuses the desktop step's title and lede.
- **The permission button says Continue, not Allow.** The browser's own Allow
  comes next, and two Allows in a row reads as being asked twice.
- **The device check is its own screen, gated by Continue.** Continue stays
  locked until the microphone is heard, so nobody records without having
  proved the mic works.
- **The question is read with the camera off,** and recording starts only on
  the candidate's tap — time to think before being watched.
- **Start recording and Stop recording are the same labelled button in the
  same place,** so the thumb never moves. A label beats a camera-app shutter
  for people who are not photographers.
- **The clip plays back afterwards.** The device check only proves the live
  preview works; playback is the only thing that proves the recording
  captured them. Recorded with MediaRecorder, falling back to the last frame
  where that API is unavailable.
- **Answers save per answer, automatically; Submit confirms the take** and is
  held until the save has landed. The Submit reading is an assumption — see
  the open questions.

Opening the question sheet or hiding the preview mid-recording repaints around
the media rather than through it, because a re-render replaces the canvas the
recorder is reading and would silently destroy the take.

### Permission dialogs

Drawn the way each platform really draws them, because this is the single
biggest drop-off point in the flow.

- **Android** asks twice. Chrome's own modal handles the site, then Android's
  runtime dialogs handle Chrome itself — camera and microphone separately, so
  a candidate who has never used the camera in Chrome sees three dialogs in a
  row. The OS ask only happens once the site is allowed. The prototype panel
  can start with the OS permission already held, which is the common case.
- **iOS** shows one Safari alert. Safari asks again on every page load unless
  the candidate sets it in Website Settings, so the question screen says so.

Three distinct blocked screens follow: the site blocked the request, Chrome
lacks the OS permission, or another app holds the camera. Each carries the
steps for that phone, and the site-blocked one draws a small mock of the
address bar showing which control to tap — Chrome's lock icon at the top,
Safari's aA at the bottom.

### What the analytics changed

Three readings from the product funnel drove this work. The figures are
internal — they live on the analytics board linked from UX-1224, not in this
public repo.

1. **Mobile is roughly half of candidate traffic**, and iPhone is the large
   majority of it. This is the main surface, not a slice.
2. **iPhone finishes at well under half the desktop rate**, and loses people
   twice: at the camera permission ask, and again between allowing the camera
   and submitting.
3. **Most of the drop at the permission step produces no event at all.**
   Explicit refusals are a small minority of it, and the instrumentation
   tracks only successes and refusals. So the largest leak in the funnel is
   currently invisible — and it is a failure, not a persuasion problem.

Three states follow from that:

- **In-app browser.** A link tapped inside Gmail, LinkedIn or WhatsApp opens
  in a webview where `getUserMedia` is often unavailable and fails without
  drawing a prompt — which is the shape of the silent drop. Caught on load,
  before the candidate spends a step on it, with the steps to reopen in the
  real browser and a copy-link fallback. Detection is a user-agent heuristic
  and is treated as one: the screen always offers "my camera works —
  continue", because a false positive costs one screen and a false negative
  costs the candidate. **Validate `detectWebview()` against real UA data
  before trusting any number it produces.**
- **Interrupted recording.** A call, a tapped notification, the screen
  locking or an app switch suspends the page and ends the camera track. Real
  `visibilitychange` and track `ended`/`mute` listeners stop the take, hold
  the last frame, and say what was kept — with "watch what we kept" to prove
  the partial answer survived. This is aimed at the second iOS gap: a large
  share of iPhone candidates who allow the camera still never submit.
- **Save that has not landed.** Saving is per answer and invisible when it
  works, which is right — and carries the risk that a stalled write lets a
  candidate walk on to question 2 with question 1 still in flight. One chip
  carries four truths (saving / saved / still saving / not saved), and
  **Start assessment is held until the write lands**. That hold is the
  feature. An offline review screen waits rather than failing, and resumes
  on reconnect.

A connection bar lives outside the rendered screen so it can appear
mid-recording without a re-render dropping the stream.

### Open questions and known gaps

- **Is the in-app-browser hypothesis right?** Segment the permission funnel
  by user agent for webview signatures (`FBAN`, `FBAV`, `Instagram`,
  `LinkedInApp`, `GSA`, `Line`). If a meaningful share of the silent drop is
  in-app, this screen is the highest-value change in the project.
- **Instrumentation gap.** Today nothing distinguishes "dialog never shown"
  from "device error" from "abandoned". Until it does, the drop at the
  permission step can only be guessed at. This is a product ask, not a
  design one.
- **What does Submit do?** The wireframe has a Submit button, and answers
  save by themselves with no upload step. The build reads Submit as "this is
  my answer", not "send it": the take is saved as a draft automatically and
  Submit confirms it. If Submit actually triggers the save, the save chip and
  the hold-until-saved logic need rethinking. Confirm with product.
- **Eight screens before the first word.** Separating the device check and the
  question costs taps on the platform that already loses people. The test
  screen's success state and the question screen are both pure Continue
  screens with no decision on them, so one of them is a candidate to fold if
  the funnel says so.
- **Retention contract.** The interrupted screen tells the candidate the
  partial answer is kept. Confirm that is what the backend actually does
  before this copy ships.
- **The microphone gate still has no bypass.** Deliberate: we genuinely hear
  nothing, so the gate's judgment is certain. The counter-argument on a phone
  is that a silent device failure traps the candidate behind a wall that
  fires no event. If that is a real risk, the shape to add after a second
  failure is not "Skip" but "Continue anyway — we'll note we couldn't test
  your mic."
- Retake policy on real questions, and the number and length of questions,
  are still open with product.

## Running it

Single HTML file, no build step. Open `async-practice-walkthrough.html`
in a browser, or use the live link above.

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
