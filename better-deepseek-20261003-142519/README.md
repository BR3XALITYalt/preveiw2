# Preview 2 Generator

A single-page web app that turns any video into the "Preview 2" format: every 1 second of input becomes 4 seconds of output, arranged as four tracks (Normal, Mari Group, Not Scary mirror, Mari Group).

Everything runs in the browser. No server, no uploads, no accounts. The video never leaves the user's machine.

## Deploy to GitHub Pages

1. Create a new repository on GitHub (public or private, both work).
2. Upload every file from this folder into the repository root. The three files you need are:
   - `index.html`
   - `README.md`
   - `.nojekyll`
3. Commit to the default branch (`main` or `master`).
4. In the repository, open **Settings > Pages**.
5. Under **Source**, choose **Deploy from a branch**.
6. Set **Branch** to `main` (or `master`) and folder to `/ (root)`.
7. Save. GitHub builds the site in about a minute.
8. Visit `https://<your-username>.github.io/<repo-name>/`.

The app is now live. No build step, no CI, no environment variables.

## Local testing

Opening `index.html` directly with a `file://` URL will not work. The browser refuses to construct Web Workers from a `null` origin, and FFmpeg needs a real origin.

Serve the folder over HTTP instead. Any of these work:

```

python -m http.server 8000

```

```

npx serve .

```

Then open `http://localhost:8000/`.

## How it works

The pipeline mirrors a desktop FFmpeg script. Each 10-second batch of input is split into 1-second slices, and each slice is duplicated into four tracks:

| Track | Video treatment | Audio pitch |
|-------|----------------|-------------|
| 1 | none | 0 semitones |
| 2 | hue shifted by 52 degrees (Mari Group approximation) | +1 semitone |
| 3 | hue shifted by 180 degrees, then split-mirror on the vertical axis (Not Scary approximation) | -2 semitones |
| 4 | same as track 2 | +1 semitone |

Track 3 uses a split-down-the-middle mirror, not a plain horizontal flip. Even slices take the left half and mirror it to the right. Odd slices take the right half and mirror it to the left. The alternation adds variation.

Pitch shifting uses `asetrate` + `aresample` + `atempo`. This is a browser-side substitute for the desktop `rubberband` filter, which is not available in the ffmpeg.wasm build. The pitch shift is correct; the formant handling is less clean than rubberband, so voices shifted by more than a couple of semitones may sound slightly chipmunky. For +/- 1 to 2 semitones it is fine.

## Limitations

- Single-threaded. Browsers cannot use SharedArrayBuffer without special headers, and we avoid depending on them for portability. Expect roughly 1x to 3x realtime on a modern laptop.
- Everything is held in memory. Files over 100 MB may crash the tab on low-RAM machines.
- The app requires a modern browser with ES module support and WebAssembly. Chrome, Edge, Firefox, and Safari 15+ all work.

## Files

- `index.html` - the entire app (HTML, CSS, JavaScript, FFmpeg wiring).
- `README.md` - this file.
- `.nojekyll` - tells GitHub Pages to skip Jekyll processing.

## Credits

Built on top of [ffmpeg.wasm](https://github.com/ffmpegwasm/ffmpeg.wasm), loaded from jsDelivr.
