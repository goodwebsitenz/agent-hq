# Fish in the water + storyboard scenes 3 and 4

Four 1920×1080 storyboard frames, rendered with the approved Pip v1 rig.
Script: `studio/pip6_board.py`.

    key3_fish_emerges.png      FULL two-shot · fish right · splash + ripples
    key4_friendly_moment.png   CLOSE two-shot · fish left · eyes meeting
    cmp_3_vs_4.png             the two side by side
    key1 / key2                re-rendered with the approved rig for consistency

---

## 1. Pip is now a versioned character, not a file that drifts

    studio/characters/pip_v1.py        Pip, version 1.0.0

It carries `VERSION`, `APPROVED_ON`, `APPROVED_TAG` and an
**`APPROVED_FINGERPRINT`** — a sha256 of the rendered silhouette at approval.
`python studio/characters/pip_v1.py` verifies the live drawing against it:

    RESULT: MATCH - approved design intact

Deliberately tested by nudging one toe radius by 0.8 units:

    before:            True
    toe_r + 0.8:       False
    reverted:          True

So the check is real, it is cheap, and it fails on a change far too small to see.
Episodes import `characters.pip_v1` — rig6 stays the engine.

Verified in this pass after all the fish work: **MATCH**, fingerprint
`b497357f…485c78`.

## 2. Fish — how the submersion actually works

A fish is not "in" water because it is drawn near water. Three things make the
read, and all three are now in `fish6`:

- **`world_masses()`** returns the fish's real paths in world space, matching a
  translate/scale draw — so the surface can be applied to the *drawn* fish.
- **`submerge()`** tints everything below the waterline *inside the fish's own
  outline* and lays a pale meniscus band across the body at the line. The
  submerged part stays faintly visible rather than being cut off, which is what
  separates "in the water" from "cropped at the water".
- **Ripples originate at the body/water intersection**, not at the fish's centre
  and not somewhere generic on the river.

The waterline used is **838**, not `env6.WATER_Y` (700). WATER_Y is the far
horizon where the river starts; the pad sits on water nearer the camera. A fish
breaking the surface at the horizon line would be floating a hundred pixels above
the actual river in front of him. Measured submersion: **28%** in scene 3,
**34%** in scene 4.

## 3. Eye contact is solved, not eyeballed

`fish6.look_at_world()` and `pip_look_at()` compute the pupil offset from the
actual world positions of the two eyes, so neither character can be pointed at
the wrong thing by hand.

Verified by comparing the solved pupil direction with the true eye-to-eye
direction:

| shot | Pip pupil angle vs Pip→fish | fish pupil angle vs fish→Pip |
|---|---|---|
| scene 3 | **43.1° vs 43.1°** | −43.1° vs −43.1° |
| scene 4 | **140.2° vs 140.2°** | −39.8° vs −39.8° |

Exact to a tenth of a degree. Travel is scaled to **0.85** of full deflection:
at 1.0 the pupil sits hard against the rim of the eye white and reads strained or
cross-eyed. 0.85 points just as clearly without the eye looking pinned.

## 4. Scenes 3 and 4 — two different shots

| | scene 3 | scene 4 |
|---|---|---|
| staging | Pip **left**, fish **right** | fish **left**, Pip **right** |
| shot | FULL two-shot — whole bodies and the pad | CLOSE two-shot — heads and shoulders |
| zoom | 1.748 | **2.055** (18% tighter) |
| Pip's head | 584 × 538 px | **698 × 623 px** |
| fish | 494 × 265 px | 540 × 333 px |
| water | splash crown + double ripple | calm single ripple |
| beat | the fish breaks the surface | friendship, eyes meeting |

Scene 4 is not scene 3 mirrored — it is a different shot size, a different beat
and the opposite staging, so a viewer reading the two boards cannot mistake one
for the other.

## 5. A real bug this pass caught

Grouping the masses for the limb-clip work shifted the list order, and the board
script was still picking the head by **index 14**. It was silently framing Pip's
*torso* as his head — keys 1 and 2 came out with a 718 px-tall "head" and the
close-up at 1.45 zoom instead of 2.16. Both scripts now take the head **by name**
(`body_parts()["head"]`). Index-based geometry access is now gone from the board
and preview scripts.

## 6. Test status
`regress6.py`: **25/26** — the single known failure remains the V3 art lock
(IoU 0.7895), still red on purpose until a new reference is cut from the approved
baseline. Pip fingerprint: **MATCH**. Audio, interpolation, camera maths untouched.

## Honest notes
- Scene 4 deliberately crops Pip below the chest. That is close-up framing, not a
  crop error; his head has 54 px of clearance and the fish is whole.
- Scene 4's fish sits 12 px from the bottom margin at 0.90 fill. It fits, but it
  is the tightest any element has been — if it reads crowded on a phone I will
  drop the fill to 0.86.
- The fish is still a one-eyed, side-on character. Eye contact works by aiming
  that single eye; there is no second eye to cheat with.
