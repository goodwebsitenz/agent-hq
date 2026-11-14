# "Who's Splashing?" — silent blocking pass

20.0 s · 1920×1080 · 24 fps · 480 frames · h264 · **no audio track at all**

This pass exists to judge motion before spending anything on sound. Four approved
shots, the approved Pip v1 rig, the current fish rig, the existing river.

| shot | time | framing | zoom |
|---|---|---|---|
| 1 | 0.00–5.00 | medium — happy float, splash #1 off-frame right, he notices | 1.350 |
| 2 | 5.00–9.50 | head close-up — splash #2, genuine startle, curiosity creeps in | 2.210 |
| 3 | 9.50–15.00 | full two-shot — the fish breaks the surface, they look at each other | 1.816 |
| 4 | 15.00–20.00 | close two-shot — shared laugh, the fish dives, he ends on the water | 1.908 |

Every framing is solved by `camera6.fit()` from the character's measured ink
bounds, the same solver the approved storyboard used.

---

## 1. Minor cleanup applied during animation

Pip's approved design is untouched: `characters/pip_v1.py` still verifies
**MATCH** against the recorded fingerprint
`b497357f…485c78` — and that fingerprint hashes the rendered silhouette, so a
paint-order or clip change could not hide behind it. Regression suite: **25/26**,
the one red still being the V3 art-lock, which is red on purpose.

Each item was measured before and after. "Looks odd" is not a measurement.

**A. Lower-eye arcs — reduced.** On the approved key-1 pose (lidLow 0.27) the
lash arc was **4 269 px** of extra ink at 4×, drawn at 1.5× ring weight across
0.94 r of a round eye: a stroke across the eyeball rather than a lid edge. Now
**914 px** (−79 %), at 1.15× ring weight over 0.86 r, with a shallower curve, and
it fades in with the lid so a light squint shows no hard line at all. The laugh
pose went from 78 px to **0** — a squint that deep no longer needs the line.

**B. Torso shading — simplified.** The contact shadow was painted per limb mass
straight onto the canvas, so the alphas accumulated: an arm is six masses, and
0.55 + 0.55·0.45 + … converged on an effectively opaque patch. Measured darkest
interior pixel **(142,196,12)** — the shade colour itself. It is now drawn
through one `saveLayer` at 0.55, so it composites **once**: darkest interior
**(148,204,14)**, shadow opacity 0.55 where it used to reach 0.81, blur 5.0 → 6.5.
That flat dark plateau is what read as a "torso gradient". Limb separation is
slightly softer as a trade — say the word and it comes back up to 0.65.

**C. Black wedges at the inner arm/hip joins — removed, and it was a bug, not art.**
The two-pass paint (every ink, then every fill) means a later group's fill covers
an earlier group's ink. That is correct for the far limbs — the torso is meant to
swallow them — but `near_arm` is last in paint order, i.e. in front, and
`near_leg`'s fill was covering its outline. The result was an isolated **864 px
pure-ink island** on each elbow, 10×9 model units, marooned in body colour and
disconnected from every other line. Exactly the "small black wedge". Far limbs and
near limbs are now drawn as self-contained ink+fill units around a two-pass core;
the islands are **gone** (the only remaining separate ink components are the
mouth and the two nostrils, which should be there).

**D. Splash overlap on the fish's face — removed.** The crown was drawn *after*
the fish, over him. It is now drawn *before* him and offset to his tail side:
**106 world units** between the nearest plume edge and his eye, verified
geometrically.

**One storyboard change, forced by motion.** Key 4 mirrored the staging (fish left
of Pip) because a still has no memory. In a continuous take that is a geography
jump — he is right of Pip in shot 3 and would be left of Pip in shot 4 with
nothing to motivate it. Shot 4 keeps him on the right and takes its difference
from shot size instead.

---

## 2. Visible problems still present

1. **No dialogue, so two beats have no motivation.** "Ooh! What's that?" and
   "Let's find out!" are absent by design; shots 2 and 3 are carried entirely by
   eyes, head and gesture. Whether the startle reads as *surprise* rather than
   *alarm* with no voice is the single biggest open question in this pass.
2. **Shot 4 is only 19 % tighter than shot 3** (zoom 1.908 vs 1.816), not the
   ~33 % the board implied. Framing the fish on the right widens the union and
   `camera6.fit()` refuses to crop, correctly. The difference is carried more by
   *what is in frame* — heads only, both bodies bleeding off the bottom — than by
   zoom. It does read as a different shot, but it is a close two-shot, not a
   two-shot of heads.
3. **Three hard cuts, no transitions** (4.96 s, 9.46 s, 14.96 s). They show as
   42.8× / 29.1× / 25.5× frame-diff spikes against a median of 1.2. Intentional,
   but the 9.46 s cut changes Pip's size a lot and may read as a jump.
4. **Splash #1 and #2 are rings only**, at the frame edge, no crown by design.
   Without sound, "something splashed over there" is unproven — a child may see
   nothing happen at 1.95 s.
5. **The 4.35–5.00 s hold is nearly static.** Only breathing and the river move.
   That is deliberate (a hold buys the next reaction) but it is the one place a
   3–8 year old may lose attention.
6. **Cutting to the close-up at 5.0 s while the splash is still ringing** is tight;
   if the audio drop lands late the beat will feel crowded.
7. **The fish's tail fan is the widest thing in the two-shots** — at the close shot
   it reaches x=1901 of 1920, 19 px from the edge.
8. **No parallax change across the cuts.** Each shot draws the same layer set at a
   different zoom; there is no new background information at a cut, which is part
   of why the cuts feel like zooms rather than moves.

---

## 3. Numbers behind the delivery

- Container re-measured on the file, not the log: **1 video stream, 0 audio
  streams**, 1920×1080, 24.000 fps, **480 frames**, **20.000 s**, h264, 5.7 MB.
- **Framing gate on all 480 frames**: Pip's head ≥16 px from the top and ≥12 px
  from each side, fish fully inside frame in shots 3–4.
- **Fish float depth 30.2 %–35.4 %** of his body height below the surface
  (186 emerged frames). Measured on the body mass only — an earlier version of
  this metric included the tail fan, whose ±15° sweep alone moved it between 24 %
  and 42 % while the body never moved.
- No black frames. Pip present in **480/480**. Fish colour absent before 10.30 s
  (max 0 px) and present through 10.90–18.50 s (min 8 876 px).
- **13 frame-diff spikes, all mapped to a written beat.** Three are the cuts, two
  the splashes, two the rise, three the laugh bounces, one the dive, two beats
  where curiosity changes the face.
- Determinism: the same pose rendered twice is byte-identical (regression suite).

---

## 4. How to read this pass

Judge: does the fish's entrance land at 10.3 s? Is the point at 12.6 s legible as
"it's you" rather than a wave? Does the ending have any charm, or does the film
just stop? Those are the questions motion answers and a storyboard cannot.

Nothing is rendered with audio until you say so.
