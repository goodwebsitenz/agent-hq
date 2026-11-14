# Pip — polish pass on the approved baseline (v1)

Two 1920×1080 PNGs. **No design changes. No silhouette changes.**
Script: `studio/pip6_preview.py`.

    PIP_pip_fullbody.png     full-body happy, polished
    PIP_pip_surprised.png    startle, bust framing, polished

The approved baseline is locked at tag `pip-baseline-v1` and copied to
`samples/pip6/approved_baseline/PIP_BASELINE_v1_*.png`. Nothing in this pass
moved a single outline point.

**Proof the silhouette is unchanged:** the regression test that compares against
the V3 art reference reports IoU **0.7897 → 0.7895** across this pass. A 0.0002
drift is anti-aliasing on the shadow. If I had resized, re-proportioned or moved
the character, that number would have moved by percent, not by rounding.

---

## 1. Full body

### The technical creases — and what they actually were

There were **two** sources, and only one of them was the one I had drawn.

**(a) The neck disc.** The rig needs a disc centred on the neck pivot so a
rotating head never opens a seam. It is a gap-filler. It was being **stroked**
like every other mass, which drew a 36-unit ink ring straight across the upper
torso on every frame — a construction artifact that had been on the character
since V6 and that nobody had isolated. Now it is drawn as a **fill with no
outline**: it still fills the gap, it no longer draws a ring.

**(b) Limb outlines crossing the body.** The arm and leg capsules were painted
across the torso, so the round end-cap of each limb drew a hard arc *inside* the
green body — the "technical-looking inner arm/shoulder crease". Limbs are now
clipped out of the torso (`ClipOp.kDifference`): each limb is painted only
outside the body, so its outline stops cleanly at the body edge, and the torso's
own outline is interrupted where the limb emerges. That is how a limb joins a
flat-vector body.

### The soft join

With the interior outlines gone, separation is now carried by **light**: each
limb's shape is re-painted inside the torso in a slightly darker green
(`#89bd0d`) at 0.55 alpha with a 5-unit blur, clipped to the torso. It reads as a
contact shadow on a rounded form, not as a line. Measured: 16.5k shadow pixels
present, and zero hard ink arcs inside the torso.

The visible hind legs and three-toed feet are untouched — knee bulge, narrowing
and toe bumps all still read, because the outer contour zigzag is unchanged.

### Happy face charm

Slight only, inside the existing design:
- eye highlights `hl1` 4.4 → **5.0**, `hl2` 2.0 → **2.3** — brighter, livelier,
  still clean
- blush 1.15 → **1.30**
- `mouthSmile` 1.45 → **1.50**, `mouthWide` 1.05 → **1.07**

## 2. Surprised

The O mouth is kept exactly as approved (`mouthO`, ellipse, no corners) and the
tongue stays off. What changed is everything that carries the *reaction*:

| channel | was | now |
|---|---|---|
| body lift `dy` | −7 | **−12** |
| squash | 1.035 | **1.07** |
| head lift `headDy` | −4.5 | **−7** |
| head angle `headTilt` | −7.5° | **−11°** |
| arms `armL/R` | 18 | **26** |
| elbows `armLEl/REl` | −15 | **−22** |
| arm reach `armLen` | 1.10 | **1.25** |
| legs splay `legL/R` | 0 | **−12** |
| `eyeScale` | 1.22 | **1.26** |
| `pupilScale` | 0.55 | **0.50** |

**Framing changed from a head close-up to a bust (0.64 → 0.52 of frame).** That
is the single biggest reason the still now reads: a head-only crop throws away
the arms, the shoulders and the lifted body, which *are* the recoil. The face
alone cannot carry a startle in a frozen image.

Measured off the delivered PNG:
- mouth **54 × 86 px**, aspect **0.63** — still the only mouth in the set that is
  taller than it is wide
- pupil area / eye-white area **0.019** (was 0.027, and 0.460 in happy)
- tongue pixels **0**
- no brow arcs — there are none in the system to reintroduce

## 3. Test status

`studio/regress6.py`: **25/26**, unchanged.

The one failure remains `rig: neutral still matches approved V3 art` — **IoU
0.7895** — which locks the character to the pre-legs V3 reference. It stays red
on purpose until a new reference is cut from the approved baseline. Everything
else passes, including *single silhouette, no gaps, no holes across 10 poses*,
which is the check that matters after clipping limbs out of the torso.

Audio, interpolation, camera maths and the attachment fixes are untouched.

## 4. Next, in the order agreed
1. fish waterline and submersion
2. fish/Pip eye contact
3. re-stage key 4
4. updated storyboard frames
5. only then animation

## Honest notes
- The shadow is a deliberate departure from pure flat colour. It is subtle and
  it is what replaced the outlines, but it is a shading element — if it reads as
  off-style at full size, turning it down is a one-number change
  (`0.55` alpha in `rig6.draw`, pass 3).
- The bust framing shows Pip's chest but not his feet. That is a framing choice
  for this one still, not a change to the character.
