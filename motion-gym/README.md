# Motion Gym

**A workout you play, not one you get through.**

Motion Gym turns a webcam into a two-player obstacle course. Stand in front of
the camera and the screen puts boxing gloves on your hands and a tracked
skeleton on your body — then obstacles come at you, and the shape of each one
tells you what to do. A hazard block means punch it. A low bar means duck under
it. A wide gate means jumping jack. You clear them with your body, and the score
takes care of itself.

It was built for my aunt, on a simple idea: the hard part of exercising at home
is rarely the exercise, it's starting. A game asks differently. There's nothing
to strap on, nothing to charge, no account and no gym — you open a web page,
step back, and play. Two people can go head to head on one camera, which turns a
workout into something you do *with* someone instead of alone.

Underneath, it uses Google's MediaPipe pose tracking to read 33 body landmarks
in real time, and measures every rep against *your own* body: a three-second
calibration takes your proportions, so squat depth and rep thresholds scale to
you rather than to some assumed average. **The camera feed never leaves the
device** — the model runs locally in your browser, nothing is uploaded, nothing
is recorded, and there's no server to send it to.

Everything is one self-contained file: [index.html](index.html).

> **Not a medical or fitness-tracking device.** It counts reps well enough to
> make a game fun, not well enough to base a training plan on. Move at whatever
> pace suits you, and skip anything that doesn't feel good.

## Run it

`getUserMedia` only works in a secure context, so `file://` **will not work**.
Serve the folder:

```bash
python3 -m http.server 8777 --directory ~/Documents/projects/motion-gym
```

Then open <http://localhost:8777> in Chrome and allow the camera.

First run downloads ~6 MB (MediaPipe runtime + pose model) from a CDN; after
that it is cached.

## How long is a game?

**5 minutes.** Waves step up every **45 seconds**, so a session runs through
six waves, then the summary screen. Both numbers are at the top of the script:

```js
waveSecs: 45,       // seconds per wave
sessionSecs: 300,   // seconds per session
```

Calibration adds about 3 seconds up front.

## How it works

Both players stand **side by side**, facing the camera. The left half of the
mirrored view is Player 01 (NOVA, teal), the right half is Player 02 (BLAZE,
coral). Obstacles rush toward you with perspective scaling, and the obstacle
**type** dictates the move:

| Obstacle | Move | Wave |
|---|---|---|
| Hazard block | Punch it | 1 |
| Low bar | Squat under | 1 |
| Wide gate | Jumping jack | 1 |
| Side wall | Dodge away from the mark | 2 |
| Star | Reach both hands overhead | 2 |
| Knee strip | High knees (4, alternating) | 3 |
| Lunge gate | Side lunge to the mark | 3 |
| Hop pads | Skater hops (2) | 4 |
| Combo pad | Land the shown punches in order | 4 |

Clearing obstacles in a row builds a multiplier up to x8.

**Keys:** `C` recalibrate · `S` skeleton · `M` mute · `P` pause · `E` end now

## Sound

All audio is **synthesised with WebAudio** — no sound files, so the page stays
a single document. Cues:

| Cue | Sound |
|---|---|
| Obstacle armed | soft tick — your "act now" timing cue |
| Clear | impact burst + blip, **pitch rises with your combo** |
| Miss | falling saw |
| Calibration countdown | tick per second, rising three-tone on lock |
| Wave up | ascending four-note motif |
| Session end | descending chord |

Browsers only allow audio to start from a user gesture, so the context unlocks
on the **Enable camera** click. The `SOUND` button in the header (or `M`)
toggles it, and the choice is remembered in `localStorage`.

## Framing matters

Stand back until your **whole body, head to ankles**, is in frame. Every
threshold is a fraction of your own measured torso length, captured during the
3-second calibration, so rep detection scales to your body and to how far away
you are. The calibration screen keeps the camera visible behind it so you can
fix your framing while it counts down, and the game warns you ("step back")
rather than silently miscounting.

Two players full-body in one frame is cramped — a wide camera and a couple of
metres of distance makes a large difference.

## Honest limits

- **This is camera AR on a screen, not headset VR.** The immersion comes from
  obstacles scaling as they approach; there is no stereo rendering or headset.
- **A single camera cannot see depth.** Jab-vs-cross is a depth distinction, so
  combos ask for your *left* and *right* arm rather than pretending to tell a
  jab from a cross. Punch shape (straight/hook) is shown but never required.
- **Detection confidence varies by move.** Squats, jumping jacks, high knees,
  overhead reaches and dodges are solid. Side lunges, skater hops and combos are
  looser, because they depend on depth and on fast airborne frames.
- **Two players who walk through each other** will swap identities at the
  crossing point — position is the only thing distinguishing them. Staying in
  your own half is rock solid (0 flips measured across fast lateral dodging).
- **Burpees are not included**, so nothing needs a special camera rig.
- The calorie figure is a rough MET-based estimate against an assumed 70 kg.
  Use it to compare sessions, not as a medical number.

## Self-test

The detectors can be measured without a camera — synthetic pose sequences are
driven through the real code. In the browser console:

```js
__MG.selfTest()      // 10 scripted reps of each move + false-positive checks
__MG.scenarioTest()  // player-identity stickiness and the punch collision gate
__MG.demo()          // frozen preview of every obstacle type, no camera needed
__MG.demo(['ICE'])   // ...or just the types you name
```

Last measured (all passing):

```
reps              squat 10  jack 10  knee 10  reach 10  dodge 10  lunge 10  hop 10  punch 10
false positives   hinge-as-squat 0   drift-as-punch 0   squat-as-punch 0
                  armsUp-as-punch 0  armsOnly-as-jack 0  oneLeg-as-knees 1 (first lift only, by design)
scenario          flipsWhileDodging 0   flipsWhileCrossing 1
                  resting glove scores: false   punching: true   off-target: false
```

Thresholds live in one `T` object at the top of the script — tune there and
re-run `__MG.selfTest()`.

## Look

Editorial/Swiss-poster styling: cream paper, heavy display type, flat shapes
with hard keylines, acid-lime accents, and pastel player panels. Nothing glows.

| Token | Value |
|---|---|
| Ink | `#171B17` |
| Paper | `#F0EDE4` |
| Stage | `#252B27` |
| Lime | `#C6FF48` |
| P1 Nova | `#18A694` on `#C5F0ED` |
| P2 Blaze | `#FF775F` on `#FFC8B9` |

## Credits

Boxing glove sprite: [Twemoji](https://github.com/twitter/twemoji) (`1f94a`),
© Twitter, Inc and other contributors, licensed
[CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/). Recolored per player
and inlined as a data-URI.

Type: [Archivo Black](https://fonts.google.com/specimen/Archivo+Black) and
[DM Sans](https://fonts.google.com/specimen/DM+Sans), both Open Font License,
loaded from Google Fonts.

Pose tracking: [MediaPipe Tasks Vision](https://ai.google.dev/edge/mediapipe)
`PoseLandmarker` v1.0.1.
