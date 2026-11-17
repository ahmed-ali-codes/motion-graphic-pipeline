# Creator Motion Graphics Pipeline

A pipeline for building "Creator Motion Graphics" kinetic-type explainer videos: a self-contained HTML animation drawn on a canvas, rendered frame by frame in headless Chrome, and encoded to a 1280x720 MP4 with synthesized sound.

This pipeline is from **Nafay 3D**: https://nafay3d.com/ (the "Creator Motion Graphics" style comes from the Nafay 3D Style Library).

## What is in this repo

| File | Purpose |
| --- | --- |
| `cmg-prompt-and-reference.txt` | The style prompt and a complete working reference animation (about 860 lines). Its line numbers matter, because the build script cuts it by line range. |
| `cmg-pipeline-kit.txt` | Setup steps, the build workflow, and the four scripts (`build.py`, `harness.js`, `render.js`, `stills.js`) stored as text blocks. |
| `cmg-example-story.txt` | A finished example of the three story modules (`style.js`, `scenes.js`, `score.js`) for a 39 second video. Use it as a reference for detail level only. Do not reuse its story. |

## How it works

1. **Storyboard.** Draft 6 to 10 beats with their timing, on-screen words, camera moves, transitions and sounds. Get approval before building.
2. **Build.** `build.py` combines the reference's helpers with your story modules and writes `animation.html`.
3. **QA stills.** `stills.js` renders PNG frames at chosen times. Check every still for overlapping or off-frame text, small labels, and factual errors.
4. **Render.** `render.js` seeks every frame (30 fps), captures the canvas, pipes the frames into ffmpeg, and muxes in the synthesized audio (`loudnorm` at -16 LUFS).
5. **Verify.** Check the MP4 duration, audio track, file size (under 100 MB), and page errors. Then compare a few frames from the MP4 against the stills.

## Requirements

- Node.js and npm
- Python 3
- Google Chrome or Chromium
- ffmpeg and ffprobe

## Setup

```bash
mkdir -p WORK/story && cd WORK
npm init -y
npm i playwright @fontsource/anton @fontsource/instrument-serif @fontsource/inter
```

Set the `CHROME` environment variable to your browser path if it is not at `/usr/bin/google-chrome`. Google Fonts may be blocked in some sandboxes. The harness injects the local `@fontsource` files instead.

## Build steps

1. Copy `cmg-prompt-and-reference.txt` unchanged to `WORK/prompt.txt`.
2. Split the scripts in `cmg-pipeline-kit.txt` into `WORK/build.py`, `WORK/harness.js`, `WORK/render.js` and `WORK/stills.js`.
3. Write your story modules:
   - `WORK/story/meta.json`: `{"title": "...", "meta": "topic · NN s", "dur": NN, "aria": "..."}`
   - `WORK/story/style.js`: the timeline `TL` (including `TL.D`), `CUTS`, `PUNCH`, `SHAKE`, `FLASH`, `LEAK` and value curves.
   - `WORK/story/scenes.js`: `background()`, `camera()`, `chrome()`, your scene functions and `SCENES`. Put `DRAWYAW_PLACEHOLDER` on its own line if you use `drawYaw`.
   - `WORK/story/score.js`: `scoreSound(k)`, with sound events driven by the same `TL` times as the picture.
4. Build: `python3 build.py WORK` writes `WORK/animation.html`.
5. QA: `node stills.js WORK 0.4 1.0 2.5 ...` writes stills to `WORK/qa/`. Review each one.
6. Render: `node render.js WORK` writes `WORK/video.mp4`.

## Rules to keep

- Every frame is a pure function of `t`. Use `CM.rng` and `CM.noise1` for randomness, never `Math.random`.
- Keep the headline text at 80 px or more, labels at 28 px or more, and key numbers at 120 px or more, so it reads on phones.
- Keep the total length at 15 to 40 seconds. The hook should land in under 2 seconds, and something new should appear every 0.5 to 1.5 seconds.
- Use invented, generic names for mockups. Do not use real logos, brands or people.
- Verify every fact and number shown. Label illustrative figures as examples, and record sources in `WORK/script.json`.

## Notes and limitations

- `build.py` pulls the reference's sections by hard-coded line ranges (for example, lines 122 to 296 for the core script and 818 to 859 for the player). Do not edit, reflow or re-save the prompt file with different line endings, or the build will break.
- The reference's `MY CONTENT` section is a placeholder. Fill it with your topic or script before use.
- The scripts are stored inside `cmg-pipeline-kit.txt`, not as separate files. Extract them before running.
- The example story is a sample for format and detail. It is not a template for your content.

## Credits

Pipeline and "Creator Motion Graphics" style: [Nafay 3D](https://nafay3d.com/).
