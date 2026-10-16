# Optional recorded sound clips (CC0 only)

Everything you hear in Ashcombe Mill is synthesised at start-up by `src/audio/` — this folder can stay empty.
If a file below exists, the sound module uses it **instead of** the matching synth layer.

**Licence: only CC0 / public-domain recordings may go in this folder.** Nothing CC-BY, nothing "free for
personal use", nothing ripped from a video. Record the source URL and licence in `docs/credits.md` anyway.

## How clips are picked up

- On first `toggle()` the module sends a `HEAD` request for `audio/<slot>.ogg`, then `audio/<slot>.mp3`
  (relative to the app's base URL). A 404 — or an HTML fallback page from the dev server — means "keep the synth".
- The first file that exists *and* decodes wins. Safari cannot decode Ogg Vorbis on older versions; supply an
  `.mp3` alongside the `.ogg` if you care about them (the `.mp3` is tried when the `.ogg` fails).
- Every clip is played as a seamless **loop** from a random offset, so cut it on zero crossings or with a short
  crossfade at the seam, and avoid one-off events (a dog bark, a car) that would be recognised on repeat.
- Use 44.1 or 48 kHz, 16-bit source, encoded at Vorbis q4–q6 or MP3 160–192 kbps. Keep each file under ~2 MB.

## Slots

| file (`.ogg` or `.mp3`) | what it should be | replaces | channels / placement | loop length | loudness of the file |
| --- | --- | --- | --- | --- | --- |
| `stream` | a gentle chalk-stream babble recorded close (1–2 m), no roar, no wind noise | stream grains + burble bed | mono; looped on the 4 stream emitters nearest the camera, spatialised | 30–60 s | −20 LUFS integrated, peaks ≤ −3 dBFS |
| `weir` | a small weir / cascade a few metres away: low-mid roar with some splash, little hiss | weir rumble, body, hiss and surges | mono; looped on 3 emitters across the weir, spatialised (distance-filtered) | 30–60 s | −18 LUFS, peaks ≤ −3 dBFS |
| `wheel` | an overshot water wheel turning slowly: wooden creaks, water pouring and spilling | wheel creaks, groans, clank, pour and spill | mono; looped at the axle, spatialised | 40–90 s (several revolutions) | −20 LUFS, peaks ≤ −3 dBFS |
| `wind` | light wind in deciduous leaves (willow, oak), gusting, **no** mic rumble | wind beds, gusts and leaf rustle | stereo; non-spatial ambience (louder in drizzle) | 60–120 s | −28 LUFS, peaks ≤ −6 dBFS |
| `rain` | soft drizzle on leaves and grass, close, no thunder, no traffic | rain patter and plinks on water | stereo; non-spatial ambience, only while drizzling | 30–60 s | −26 LUFS, peaks ≤ −6 dBFS |
| `birds-day` | English countryside birdsong, daytime: blackbird, robin, chiffchaff, wood pigeon | the midday and golden-hour singers | stereo; non-spatial, plays for `midday` and `golden` | 90–180 s | −28 LUFS, peaks ≤ −6 dBFS |
| `birds-dawn` | a dawn chorus: many overlapping species, dense | the dawn singers | stereo; non-spatial, plays for `dawn` | 90–180 s | −26 LUFS, peaks ≤ −6 dBFS |
| `night` | a summer night: crickets / bush-crickets, the odd tawny owl far off | owls and crickets | stereo; non-spatial, plays for `moonlit` | 90–180 s | −32 LUFS, peaks ≤ −8 dBFS |

Mood crossfades (~2.5 s) and the drizzle mix apply to clips exactly as to the synth layers.

## After adding a clip

Run the audio QA and compare its per-position levels with the synth baseline:

```sh
node tools/serve.mjs 4502 &          # if it is not already running
node tools/audio-test.mjs --stems    # renders docs/audio/*.wav, prints peak / RMS / hiss check per scene
```

Synth baseline (solo layer RMS, dBFS, from `docs/audio/levels.json`): weir −20 by the weir (−27 by the wheel) ·
wheel −21 by the wheel · stream −29 by the weir or wheel · birds −28…−30 at dawn · wind −38…−45 calm, −31…−35 in
drizzle · rain −35 · crickets −36…−40 · owl calls −38…−43 · pond lapping −40 on the shore (−33 in drizzle).
Whole mix: ≈ −19 by the weir or wheel, −28…−36 in the meadow, peaks ≤ −3.2 dBFS.
If the clip lands far from its baseline, trim it with `LEVELS.<layer>` in `src/audio/audio.js` (or re-export the
file at the loudness above). Nothing may peak above −1 dBFS in the mix and no layer may add a flat 6–12 kHz floor —
the QA fails on either.
