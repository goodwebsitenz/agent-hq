# Pip — silhouette and expression corrections (approval pass 1)

Two 1920×1080 previews, drawn by `rig6` / `expr6` / `camera6`.
Script: `studio/pip6_preview.py`.

    PIP_pip_fullbody.png     full body, happy, 80% of frame height
    PIP_pip_surprised.png    head close-up, startle, 64% of frame height

---

## 1. Pip's silhouette — he had no legs

**Root cause.** The left/right *masses* were fine, the leg geometry was not. The
old rig drew one 22×20 ellipse at (44, 124) and a shin from there. Pip's torso is
58 units wide at that height, so the entire hind leg sat **inside the torso
fill**. The limb was drawn, every frame, and covered by the body. Same class of
bug as the invisible brows: geometry present, geometry occluded.

**What it is now.** Each hind leg is built as capsules between real joints:

    hip (34, 96) -> knee (86, 108), r 22 -> 15     thigh, swings OUTSIDE the body
    knee -> ankle (74, 146), r 15 -> 8.5           shin, drops and angles back IN
    ankle -> three toes at (98,149) (78,157) (58,153), r 7.5, webbed fan

The outline now reads **torso 58 → knee 101 → ankle 82 → toes 105**. That zigzag
is the whole trick: the knee must be the widest point of the lower body and the
shape must pinch back in before the foot, or it looks like a flipper hanging off
the side. Measured model silhouette is now 211 wide × 158 tall, where it was
131 × 156 — the character finally has a leg span.

Arms were also re-cut: radii 11/10/9/8 → 9.6/7.4/6.0/7.4 and the chain moved
forward, so the arm is a limb rather than a stub welded to the torso. Three
fingers, 11.5 units, spread 6.2.

**The seam problem nobody had noticed.** Because every ink stroke is painted
before every fill, a limb overlapping the torso has *no line where it meets it* —
the torso's fill covers the limb's inner stroke. In silhouette the leg read fine;
in colour the character was one blob with bumps. First attempt was a hand-placed
crease arc; it landed on the shoulder and floated. The fix is to stroke **the
thigh capsule's own outline, clipped to the torso** — it cannot land anywhere
except exactly where the limb crosses the body, and it fades out at free air.

## 2. Eyes — brows removed entirely

The brow arcs are gone from the expression system. At this head size and ink
weight they sat inside the eyeball and read as thick lines drawn on the eye, which
is precisely what they were. `brows()` now returns an empty list; `browRaise` and
`browAngle` remain in POSE as harmless no-ops so no existing pose or action raises.

Emotion is carried by things that belong to a frog's eye:
- **`eyeScale`** — widens the whole eye (surprise 1.22).
- **`lidLow`** — lower lid rising (happy 0.08, laugh 0.48).
- **`lidHigh`** — NEW: a subtle upper eyelid, the same clip mechanism as a blink
  held part-way. Available for sleepy/concerned without faking a brow.
- **pupil size and direction**.

Happy and neutral now use **fully open eyes**. The old 0.22 lower lid was enough
to read as sleepy at this head size — that was the actual complaint, not the
mouth.

## 3. Surprise — measured, not asserted

The old "surprised" was a lip curve with `mouthSmile` 0.25 and `mouthWide` 0.80:
a wide D with upturned corners, i.e. a grin. A lip curve built from two quadratics
**always leaves two corners**, and two corners always read as a smile, no matter
how open the mouth is.

- **NEW pose channel `mouthO`** — the mouth becomes a closed ellipse, which has no
  corners at all. `surprised` uses `mouthO 1.0`, `mouthWide 0.40`, `mouthOpen 0.55`.
- Tongue is off (it is an explicit channel; it defaults to 0).
- Eyes 1.22, pupils 0.55.
- Head tilts back 7.5°, body lifts and recoils.

Measured off the delivered PNGs, not the source:

| | surprised | happy |
|---|---|---|
| pupil area / eye-white area | **0.027** | 0.460 |
| mouth open area | **71 × 113** (aspect 0.63, taller than wide) | 279 × 88 (aspect 3.17) |
| tongue pixels | **0** | 0 |
| dark arcs inside the eye | **none** | none |

Pupils are 17× smaller relative to the eye than in the happy pose, and the mouth
is the only shape in the set that is taller than it is wide.

## 4. Test status — one deliberate failure

`studio/regress6.py`: **25/26**.

The failure is `rig: neutral still matches approved V3 art (IoU >= 0.96)` —
**IoU 0.7897**. That test locks the silhouette to the old V3 reference image, and
the silhouette has been changed on purpose: he has no legs in V3. The check is
doing its job — it caught that the character changed. It is left failing rather
than quietly re-baselined, because re-baselining it now would be exactly the kind
of "tests pass so it must be good" that this brief warns about. It gets a new
reference the moment the new silhouette is approved.

Everything else still passes: 12+ actions, easing/springs, closed eyes leave no
white sliver, half-closed lid stays inside the eye, audio −16 LUFS / −2.6 dBFS /
dialogue +10.9 dB over music, no teleports, jaw shut in silence.

Untouched: audio system, interpolation, camera maths, attachment fixes, QC scripts.

## 5. Not done yet — queued for after approval
- **Fish waterline** (priority 4): the fish will be clipped by the water surface
  and its ripples will originate at the body/water intersection. Not built this
  pass because it only shows in the two-shot frames, which are on hold.
- **Frame 4 restaging** (priority 5): facing each other, eyes meeting.
- Re-baseline the V3 silhouette test.
- Only then: re-render the remaining keys, and only after that any video.

## Honest notes
- The full body is staged on a plain field, not the river. A silhouette review on
  a busy background is a composition review, and would hide exactly the problems
  this pass is for.
- The recoil is a motion cue. In a still, the face carries the startle; the body
  recoil will read properly on the frame it happens.
- The head-to-body junction is unchanged. It reads as a smooth single form with no
  neck, which is the character's design. If he wants an actual neck taper, that is
  a head-profile change and I would want it approved separately.
