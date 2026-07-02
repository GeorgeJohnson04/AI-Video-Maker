# 🎬 AutoVideoStudio

Turn a written script into a finished, 10-minute-plus YouTube video with **one command**.
It draws its own crude MS-Paint stick-figure art, generates a British voiceover, optionally
adds music + sound effects, and assembles everything into `final.mp4` — fully editable.

**Built to run with ZERO API keys.** Images are drawn locally; the voice uses free Microsoft
online TTS. Add keys later only if you want premium options.

---

## ⚡ Quick start

```bash
cd "AutoVideoStudio"
python -m pip install -r requirements.txt          # one time
python make_video.py scripts/dancing-plague-1518.md
```

Output lands in `output/dancing-plague-1518/`:

| File | What it is |
|---|---|
| `final.mp4` | the finished video (1080p) |
| `images/` | one MS-Paint PNG per scene — **edit or replace any of them** |
| `audio/` | per-line voiceover + the mixed track |
| `captions.srt` | subtitles, ready to upload to YouTube |
| `report.md` | per-scene timing + which engines were used |

The included **Dancing Plague** script already renders to **~10:34**.

---

## ✍️ Writing a new video (this is the whole workflow)

Make a file in `scripts/`, e.g. `scripts/my-topic.md`:

```
title: My Video Title
voice: en-GB-RyanNeural
music: assets/music/bed.mp3        # optional

=== scene
narration: The exact sentence the narrator says here.
image: what the picture should show (content only — the MS-Paint style is automatic)
sfx: thunder rumble                # optional short keyword

=== scene
narration: The next line, which becomes the next image.
image: a stick king on a wobbly throne
```

Rules:
- Each `=== scene` = **one image**, shown for exactly as long as its narration. Keep each
  line to ~1–2 sentences so images change every ~5–10s (great for retention).
- `image:` describes content; if you omit it, the narration is illustrated instead.
- A silent beat: use `silence: 1.5` instead of `narration:`.
- A dramatic breath: add `pause: 0.7` to a scene — the narrator pauses after that line
  (and the TTS restarts there, which keeps long reads sounding fresh). Use at act breaks
  and punchlines; write narration conversationally (contractions, "..." and "—" for pacing).

Then: `python make_video.py scripts/my-topic.md`

**Two videos a day:** drop several scripts in `scripts/` and run `python make_video.py --batch`
— it renders every script that doesn't already have a `final.mp4`.

---

## 🎨 Images — three engines

Set `image_engine` in `config.yaml` (or `--image-engine`):

- **`paint`** *(default)* — the built-in MS-Paint renderer. Crude wobbly black outlines, flat
  primary colors, stick figures, white background. No key, no cost, no internet. It reads each
  scene description and composes stick figures, crowds, signs, speech bubbles, suns, scrolls,
  question marks, etc. **Don't like one frame?** Just open `output/<video>/images/img_NNN.png`
  and redraw it yourself, or edit the `image:` line and re-run with `--fresh`.
- **`openai`** — real `gpt-image-1` ("ChatGPT Image 2") using the exact MS-Paint prompt spec in
  `avs/prompts.py`. Needs `openai_api_key`. Costs money per image.
- **`placeholder`** — text-on-white frames for previewing timing only.

---

## 🔊 Voiceover — two engines

- **`edge`** *(default)* — free Microsoft online TTS, no key. British voices:
  `en-GB-RyanNeural` (calm narrator), `en-GB-ThomasNeural`, `en-GB-SoniaNeural` (f).
  `tts_rate: "-10%"` slows it for cinematic gravitas (and helps clear the 10-min mark).
- **`elevenlabs`** — premium "Christopher"-style voice. Set `elevenlabs_api_key` +
  `elevenlabs_voice_id` (copy the id from your ElevenLabs Voices page).

---

## 🎵 Music & 🔉 sound effects — built in, no downloads

By default the pipeline **synthesizes its own audio** (`avs/synthsfx.py`, pure numpy): a dark,
loopable ambient **music bed** plus a recipe-built **sound effect** for every `sfx:` keyword
(boom, thunder, church bell, heartbeat, whoosh, crowd, coin, boing, buzzer, choir, drip, spark,
record scratch, …). No keys, no licensing, no internet. The lo-fi synth character suits the
crude art. Levels are auto-limited so effects never clip the voice.

Resolution order (per `sfx_engine: auto`): **your local files → Freesound (if token) → built-in synth.**

- **Want real recordings instead?** Drop files in `assets/sfx/` named after the keyword (these
  always win), or add a free `freesound_token` to `config.yaml`.
- **Music:** in a script, `music: synth` (built-in bed), a file path, or a keyword (with token).
  Looped, trimmed to length, and ducked low under the voice (`music_volume`).
- Turn it all off with `sfx_engine: off`.

---

## ⚙️ Useful flags

```
python make_video.py scripts/x.md --limit 5          # quick 5-scene test render
python make_video.py scripts/x.md --fresh            # wipe & regenerate (use after changing engines)
python make_video.py scripts/x.md --image-engine openai
python make_video.py scripts/x.md --no-sfx
python make_video.py --batch                         # render all new scripts
python selftest.py                                   # verify the pipeline end-to-end
python art_preview.py scripts/x.md                   # render just the images to output/_arttest
```

---

## 📈 Why it's tuned for the algorithm
- A new image every ~5–10s keeps visual novelty high (retention).
- The example script opens with a 30-second hook, uses a 3-act structure, and plants
  curiosity loops that only pay off at the end.
- Slightly slowed narration + breathing-room gaps clear the 10-minute watch-time threshold
  without padding.

## 🧱 How it works (for the curious)
`make_video.py` → `avs/script_parser.py` (script→scenes) → `avs/images.py` + `avs/paint.py`
(draw frames) → `avs/tts.py` (voiceover) → `avs/sfx.py` (effects/music) → `avs/assemble.py`
(ffmpeg: per-scene timing, audio mix, slideshow, mux). ffmpeg is bundled via `imageio-ffmpeg`
— nothing to install by hand.
