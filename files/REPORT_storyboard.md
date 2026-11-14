# Storyboard keys — "Who's Splashing?" re-shoot
Four 1920×1080 PNGs, no video. Produced by `studio/pip6_board.py`.
Pip drawn by `rig6`, fish by `fish6`, river/pad by `env6`, framing by `camera6`.
No key is a one-off drawing — each is a frame the episode renderer can produce.

## What the four keys are

| # | Beat | Framing | Pip's head |
|---|------|---------|-----------|
| 1 | Happy float downstream | medium, subject on the right third, 0.68 of frame height | 104 px clear of the top |
| 2 | Surprise at the splash | head close-up, 0.665 of frame height | 88 px clear of the top |
| 3 | The fish breaks the surface | two-shot, 0.92 of frame height | 50 px clear of the top |
| 4 | Shared happy moment | tight two-shot, 0.97 of frame height | 22 px clear of the top |

## What was actually broken, and the root fix

**1. Framing was guessed, not measured.**
`camera6.framing()` solved the camera against a frame centre of 540 px while the
renderer pivots at 620 px (`apply_cam`). Every preset therefore landed `80 × z`
pixels below where its own documentation said it would — 196 px on a 2.45 zoom.
Fixed by exporting `camera6.PIVOT` and solving against the renderer's real pivot.

**2. Nobody measured the character.**
Presets assumed "Pip is about this tall". After squash, head rotation and head
offset, he never is. `camera6.fit()` now takes the character's real ink bounds —
read from the actual mass paths via `rig6.probe()` — plus the lily pad or the
fish, and solves zoom and focus so the subject hits a chosen size with a
guaranteed margin. It **raises** rather than silently cropping. All four keys
report their measured screen boxes and margins (see run output).

**3. Brows have been invisible since V6.**
`brows()` drew the arcs at `R = eye_r + ring*0.4 = 18.7`, which is inside the
eye's own ink ring stroke (14.4–20.6). They were painted, and could not be seen.
A cartoon face whose mouth and eyes change but whose brow never appears reads as
"pleasantly surprised" no matter what the mouth does — that is the mechanical
cause of "expressions look sleepy". Brows now sit inside the eye white, between
the rim and the pupil, clipped to the eye so they cannot escape, and they clear
the ring as well as the rim.

**4. The tongue fired automatically.**
Any wide jaw opening drew a pink tongue, so a startle came out as "excited, tongue
out". Tongue is now an explicit channel (`rig6.POSE["tongue"]`, default 0) and is
only drawn where an expression asks for it (`laugh`).

**5. "Surprised" was a wide D, not an O.**
`mouthSmile` 0.25 still tipped the corners up, and `mouthWide` 0.80 made the
opening wider than tall. Now `mouthSmile 0.0` (flat top lip) with `mouthWide 0.62`
and `pupilScale 0.56` against a 1.22 eye — measured on the render, the open mouth
is 240×121 px where the happy smile opening is 377×117 px, and pupil-to-white
area is 0.03 against 0.43. That is a startle, not a grin.

**6. The fish could only ever look away.**
`fish6` is modelled nose-right; in every two-shot Pip is on the left, so the fish
had no way to face him and no eye contact was possible. Added `mirror` to
`fish6.draw()`, plus a happy squint (`lidLow`) and blush, so he can answer Pip.

**7. Two staging errors caught in the stills.**
The splash under the fish was placed 100 px *below* the waterline — it read as a
cloud behind him rather than water he had just burst through. The surprise
close-up left a dead sheet of background behind his head. Both fixed: the crown
now breaks at the surface under the fish, and a foreground reed plus an in-frame
splash fill the right of the close-up.

## Honest notes
- The surprise close-up is the least strong of the four as a *still*. A startle
  is a 6-frame event in motion; frozen, it can read as an open mouth. In motion
  the eyes widen 0.02 s before the jaw drops, which is what sells it.
- Key 2 deliberately bleeds Pip's body off the bottom edge. The head has 88 px of
  clearance and is complete; only the body below the chest is cut, which is
  normal close-up framing, not a crop error.
- Keys 3 and 4 are the same staging at different tightness. They are meant to be
  — they are consecutive beats — but if the fish reads as a duplicate in both,
  key 4 should become an over-Pip's-shoulder on the fish instead.
- Backgrounds are flat vector with no depth of field. Empty background can only
  be filled with staging (reeds, pads, leaves, dragonfly), not with focus.

## Verification
- All four PNGs are exactly 1920×1080.
- Head clearance from the top edge: 104 / 88 / 50 / 22 px. Nothing touches a side edge.
- `camera6.fit()` raises on any crop, so a passing run *is* the proof.
- `studio/regress6.py`: 26/26 checks still pass after the rig and expression
  changes. Audio fixes untouched.
