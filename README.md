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
    -> practice question (with a short countdown) -> practice recording
    -> practice review -> countdown -> question 1 (multimedia) -> review
    -> question 2 (video) -> review -> Finish -> saving -> score -> feedback -> thank you

plus the states a phone produces: five blocked states, an interrupted recording, a weak
or lost connection, a save that has not landed, and a reload.

The structure is the approved wireframe. The V8 Test Taker Experience mobile frames
and the Proctoring Attendee frames are the style and layout reference. It is built
on the real Tessera components (`window.TesseraDS`); only the simulated browser
chrome, the browser's own permission dialogs, the sample camera picture and the two
error illustrations are drawn by hand.

### The frame

Every screen is the Tessera top bar, a body with 16px padding all round, and a **fixed
footer that holds every call to action at full width**. The two camera screens (the mic
test and recording) keep the top bar and footer **solid, at full opacity**; the camera
fills the space between them, with the question or device card floating inside it.

### Two question types, both answered on video

There is no multiple choice. Both types end in the same record flow.

- **Video question.** The question is on screen, a preparation countdown runs, then
  recording starts by itself (or the candidate taps Start recording early).
- **Multimedia question.** Someone asks the question in a short video with captions
  inside it; no transcript or soundwave. Start recording stays locked until the video
  ends, then the preparation countdown runs and they answer on video.

Both types, and the practice question, use one grey card that fills the screen down to
the footer: the header ("Question n of 2", or "Practice question" with a Not scored chip),
a divider, the question as text or as the video, then plain steps in the order things
happen: the multimedia question starts with "Play the video and watch the whole
question"; every question then gives the prep time (recording starts on its own) and the
answer time and retakes.

Secondary text on the candidate screens (lines under titles, bullets, captions, pending
steps) is `text-tertiary` (zinc-700) rather than `text-body` or `text-subtle`, which read
too light on a phone. Titles and key words stay `text-primary`. The steps never change while the screen is open, so the
card does not reflow when the video ends. Rules come first
because the prep countdown starts the moment the screen opens, so the candidate needs
them before they read. The review uses the same card with the recorded answer below the
question. Videos use the web player's controls: a track, then back 5s, play or pause,
forward 5s, volume and the time (Tessera `RtIconButton`). Long questions end in an inline
Read more / Show less. There is no report icon; that belongs to the report flow.

The assessment card, the timer row, the review and the wrap-up follow the Test Taker
file. The timer is Tessera's `CandidateTimer` in its own row under the top bar, with
digits visible by default and the "Hide timer" switch on the component's own terms.

### Decisions

- **Same order as web.** A candidate practices in the sequence they will meet for the
  scored questions, so the practice never teaches a different order.
- **The question comes first, then the camera.** Stop recording sits where Start
  recording was.
- **Preparation countdown** is 10 seconds by default, with two treatments to compare
  from the prototype panel: the button fills as time runs out, or a bar above the button.
- **The practice question keeps the countdown** (on by default; the panel can turn it
  off). Practice exists to rehearse the scored question's mechanics, and recording
  starting by itself is the one thing that surprises people in the real thing; rehearsing
  it while nothing is scored is the point. The cost is a little pressure on first
  contact, which "Not scored" and unlimited re-recording absorb.
- **Device selection** uses the Test Taker file's field (a fixed 184px): the device icon inside the
  field, the device name and a chevron, with a status chip beside it. Tapping it opens a
  bottom sheet of options (Tessera `Overlay` + `RadioGroup`) rather than a desktop
  listbox; choosing applies it and closes. The sheet dims the whole page, runs to the
  bottom edge with the Android gesture bar inside it, eases in over 300ms with Tessera's
  ease-out and animates out before it unmounts. The sheet is a new pattern: Tessera's
  Dropdown panel is built for a pointer and a wide screen. In production a native
  `<select>` styled the same way would hand the choice to the phone's own picker.
- **While answering**, the overall assessment timer stays in the top bar and the answer
  time sits on the camera (no "left" label). Hiding the timer's digits keeps the row's
  height; the switch centres in it.
- **Centred text** puts each sentence on its own line (loading, countdown, the player's
  messages). Allow access is a title, one line and two bullets.
- **Wrap-up**: saving moves straight on to the score (no separate saved step). The grey
  card runs down to the footer on every screen. When the score appears the bars fill up
  one after another and confetti bursts; the share options sit left-aligned under the
  certificate. The Storyset illustrations (rocket, feedback) have slow ambient motion.
  All of it is skipped under reduced motion.
- **Device fields** are a fixed 184px, so they don't change width with the status chip.
  A silent microphone shows its message in error red with a red "No sound" chip.
- **Stop recording saves the answer**, with the web flow's copy: a spinner in the player
  while it saves, then "Your response has been saved successfully." over the picture,
  then playback. A failed save says so in the player with Try again. There is no Submit
  and no upload step, and Next, Finish or Start assessment waits for the save. Retake
  asks first ("Your current answer will be deleted if you choose to retake it"), in
  Tessera's Modal contained to the phone.
- **Finish goes straight to saving.** No confirm screen.
- **Network messages are broadcast banners** under the top bar: "No internet detected"
  (red), weak connection, back online, and the welcome-back note after a reload. After a
  long loss of connection (15 seconds here) the page is replaced by an error screen laid
  out like the Proctoring Attendee one, with Retry.
- **An interruption loses the answer in progress.** A call, a locked screen or another
  app taking the camera ends the recording; the candidate checks their camera and
  answers that question again. Answers saved earlier stay saved. A reload behaves the
  same way: onboarding stays done and answered questions stay answered, the timer
  carries on, and the current answer starts again. Progress is kept in `sessionStorage`
  where the viewer allows it. After a reload the browser asks for the camera again.
- **Blocked states** follow the Proctoring Attendee screen: a picture of a phone with
  the dialog, the title, then one bulleted list: what is wrong, then **how to fix it in
  this browser** (Chrome and Safari have different steps). Retry and the help link sit
  centred at the foot. Retry asks again from the blocked screen itself, so a candidate who
  fixed the setting goes straight on to the device check rather than back through the
  primer; the microphone help screen's Retry returns to the device check. The
  illustration sits in a box as wide as the text, with 16px inside it, matching the 16px
  gap to the title. The network error and interrupted screens use the same layout.
  The interrupted screen's action is simply "Try again". There are seven:
  the site is blocked, Chrome or Safari itself lacks permission, the camera is in use,
  no camera is found, no sound from the microphone, and two that catch the funnel's silent
  drop (below). There is no "Try with another
  device".
- **Alvy opens as a full-page overlay** under the top bar, closed with the X, so the
  timer and the bar stay visible. It offers topics, then answers with a short article.
- **The silent drop gets two states.** In the funnel, most candidates lost at the camera step
  leave no event: no allow, no deny, no error. Two causes are covered:
  - *The browser has no camera access*, often because the link opened inside another
    app (copy names no apps). Production checks the browser itself on load
    (`navigator.mediaDevices?.getUserMedia`), before the primer, rather than guessing from
    the user agent, which was the objection to the earlier in-app browser screen. The
    screen says to open the link in Safari or Chrome, with **Copy link** as its action.
  - *The request never appears.* Continue shows "Waiting for your browser…"; after 10
    seconds with no answer the candidate sees "We didn't get a response" with the fix and
    Retry.
- **Before the first question**, the countdown says to turn on Do Not Disturb, because a
  call or notification on a phone stops the recording and costs the answer.
- **A question video that fails to load** shows "The video didn't load" with Try again
  inside the player, instead of leaving Start recording locked for good.
- **The clip plays back afterwards.** The device check only proves the live preview
  works; playback proves the recording did. Recorded with MediaRecorder, falling back
  to the last frame where that API is unavailable.

The wrap-up is the Test Taker file's own: "Saving your assessment" then "saved
successfully", the score with its bar graph and certificate, feedback, and thank you.
The illustrations, bars, certificate and rating faces are exported from that file. The
score numbers (32/40 and the four bars) are the file's sample figures, not computed from
the answers.

Opening Read more or hiding the preview mid-recording leaves the media alone. The video
and canvas are children of components that stay mounted across repaints, so a repaint
cannot replace the canvas the recorder is reading.

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
  error" from "abandoned". The events to add: permission requested; permission result
  (granted, denied, error name, timed out); camera API missing; practice completed;
  recording interrupted (reason); answer save failed or retried; connection lost
  (duration); resumed after reload. Until they exist, the effect of this redesign can't be
  measured. This is a product ask, not a design one.
- **Real values.** The preparation time (10 seconds here, 5 to 30 in the panel), the
  answer limit (2 minutes), the assessment length (20 minutes), the passing score and the
  point at which a lost connection becomes the error screen (15 seconds here) are
  placeholders to confirm.
- **Retention contract.** The interrupted screen says the answer in progress is lost and
  earlier answers are safe. Confirm that is what the backend does before the copy ships.
- **The microphone gate still has no bypass.** Deliberate: we genuinely hear nothing, so
  the gate's judgment is certain. The counter-argument on a phone is that a silent device
  failure traps the candidate. If that is a real risk, the shape to add after a second
  failure is "Continue anyway, and we'll note we couldn't test your mic."
- **Touch targets.** Tessera's Button is 36px tall. Full-width calls to action make them
  easy to hit, and 36px passes WCAG 2.2 AA, but it is under the 44pt iOS and 48dp Android
  guidance. Worth raising with the design system team.
- **The red dot on the camera test** follows the wireframe, but reads as "recording". If
  candidates mistake it for that, label it.
- **Rating buttons, the score bars and the certificate** are exported from the Test
  Taker file, not Tessera components. They are flagged as new patterns.
- **The wrap-up illustrations** (saving, feedback, thank you) are exported from the same
  file and sit in `tessera/assets/figma/`, which is not committed to this public repo.
- **Timer label.** Figma swaps the switch label to "Show timer" when the digits are
  hidden; Tessera keeps it fixed as "Hide timer" because the swapped label reads as
  "Show timer, on". The prototype follows Tessera.
- **The real camera path is untested here.** The recorder, the level meter and the
  interruption listeners were exercised against the drawn sample preview, because the test
  machine has no camera. Run it once on a real phone.

## Running it

No build step. Open `async-practice-walkthrough.html` in a browser, or use the
live link above. The mobile file also needs the Tessera runtime beside it, in
a `tessera/` folder (tokens, bundle and font). It is not committed to this
public repo; add it with the design-system skill's scaffold script, which
leaves `index.html` alone:

    python3 ~/.claude/skills/tessera-design-system/scripts/scaffold.py . --assets logos

Opened from disk it uses the real camera and microphone. Where those are
blocked (for example inside an embedded viewer) it falls back to a drawn
sample preview so the flow still demonstrates.

The frame is fixed at 1440×960 and scaled to fit the window. Prototype
controls sit in the panel on the right — jump to any step, switch
variant, or force a blocked state.

The mobile file scales a 360×800 (Android) or 390×844 (iPhone) frame the same
way, and draws the browser's own chrome because the recovery screens point at
it. Its panel can switch device, force any permission outcome and any blocked
screen, drop the connection, make the save slow or fail, change the
preparation countdown style and time, and interrupt a recording in progress.

The wrap-up illustrations are exported from the Test Taker file into
`tessera/assets/figma/` and are not committed either; without them the
wrap-up screens show empty frames.
