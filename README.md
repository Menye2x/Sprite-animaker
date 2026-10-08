# Sprite Sheet → GIF / WebP / Video

A single-file, browser-based tool that turns a sprite sheet (the tile-style animation assets commonly used in MMORPGs and 2D games) into a playable animation, then exports it as an animated **GIF**, animated **WebP**, or **video**.

No build step, no backend, no install. Everything runs locally in the browser and your images never leave your machine.

## Concept

1. **Import** a sprite sheet image.
2. The tool **detects and splits** the frames automatically (or by a manual grid).
3. The frames are **played back** as one animation, with a live timeline.
4. Fine-tune the result (canvas size, alignment, per-frame offset and enhancement, upscaling).
5. **Export** to GIF, animated WebP, or video, with an estimated file size shown beforehand.

## Quick start

1. Open `sprite-animator.html` in a modern browser (Chrome, Edge, or Firefox recommended).
2. Drop in a sprite sheet, or click **Use sample** to try the built-in one.
3. Adjust the settings on the left, then export from the **Export** section.

The page loads one external script, [pica](https://github.com/nodeca/pica) `10.0.3` from jsDelivr, used for high-quality resampling. If it cannot be loaded, the tool falls back to the browser's high-quality canvas scaling.

## Features

### Import
- Drag and drop, file picker, or paste from the clipboard (Ctrl+V).
- Built-in sample sprite sheet for trying things out.
- Transparent PNGs and images with a solid background are both supported.

### Frame detection
- **Automatic mode**: finds frames by scanning rows and columns for content separated by empty gaps.
  - The background is taken from the alpha channel, or from the top-left pixel color when the image is opaque.
  - **Background tolerance** and **Minimum gap** sliders control how strictly frames are separated.
  - Frames are ordered left to right, top to bottom.
- **Manual grid mode**: set columns and rows for sheets with no gaps between sprites. Empty cells are skipped.
- After automatic detection, the grid values are pre-filled as a hint for switching to manual mode.

### Split view
- **Show split area** displays the original sprite sheet with a numbered box over every detected frame.
- Enabled frames are drawn solid, disabled frames dashed, and the current or selected frame is highlighted.
- Click a box to select that frame.

### Animation and timeline
- Play and pause, adjustable **FPS** (1–60).
- Running timeline: the playing frame is highlighted and scrolled into view, with a progress bar and a "Frame n / total" label.
- Per-frame **On/Off** toggle to include or skip frames.
- Click a frame to select it (this pauses playback) and inspect or adjust it.
- The preview draws a single **frame boundary** around the combined maximum width and height of all frames, so it does not change size from frame to frame. It can be toggled off.

### Canvas per frame
- **Auto-fit all frames**: the canvas is sized automatically to fit every frame, including margins.
- **Custom** canvas width and height.
- **Padding** around the animation.
- **Frame alignment**: bottom-center (good for walking characters), center, or top-left.

### Selected frame
- **Margin** (top, bottom, left, right, in pixels) adds a gap around a single frame. **Unify** links the four values so typing one sets all four to the same number. **Apply margin to all frames** copies it to every frame.
- **Enhance 15–200%** (100% = no extra change) for one frame. It stacks on top of the global Enhance level, so the effects multiply (for example 150% global × 150% on a frame = 225% total). The frame's size never changes.
- **Apply enhance to all frames** copies the selected frame's enhance level to every frame.

### Enhance and upscale (pica)
- **Enhance level (all frames)** from 15% to 200%, default 100% (original). Lowering it reduces quality (less detail); raising it enhances quality with Lanczos resampling and sharpening. Enhance and sharpen never change the frame boundary or canvas size.
- **Uniform upscale** of 1×, 2×, or 4× for all frames, resampled with Lanczos.
- **Sharpen strength** slider (unsharp mask, applied when enhancing; not applied when reducing).
- Defaults aim for HD-quality output: smooth scaling and 2× upscale.
- Turn on **Pixel art (sharp scaling)** to keep hard pixel edges instead.

### Export
- **GIF**
  - Transparency support.
  - A separate color palette for each frame, plus optional Floyd–Steinberg **dithering** for smoother gradients.
  - Sprites with 255 colors or fewer keep their exact palette.
  - Note that GIF is limited to 256 colors per frame.
- **Animated WebP**
  - Lossless frames with transparency, stored as full keyframes (no blending) so transparent areas never leave ghosting.
  - If the browser's encoder drops transparency, the export stops with a clear message instead of producing a wrong file.
  - Requires a browser that can encode WebP (Chromium-based browsers and Firefox). Safari cannot.
- **Video**
  - MP4 when the browser supports it, otherwise WebM.
  - Recorded in real time, so export takes about as long as the animation plays.
  - Always uses a solid background color (video does not support transparency).
  - Bitrate scales with resolution and FPS.
- Shared options: **Scale** (1–8×), **Background**, and **Loop** (1×, 2×, 3×, or infinite).
  - **Background** is a single setting (transparent or a color): it is shown in the preview and used by every export. GIF and animated WebP keep transparency; video always uses the color.
  - A video file cannot loop by itself, so *Infinite* exports a single cycle to loop in your player.
- **Estimated file size** for each format, updated live, along with the output dimensions, frame count, and duration.
  - GIF and WebP sizes are estimated from sample frames; the video figure is an upper bound.

### Interface
- Settings are in collapsible sections on the left, so the view stays compact.
- The preview and timeline sit on the right and scale to fill the available space.
- Light and dark themes follow the system setting.

## How big will the output be?

```
final size = canvas size × uniform upscale × export scale
```

For example, a 64×64 canvas with 2× upscale and 1× scale exports at 128×128 px. Large combinations (such as 4× upscale with 8× scale) get big quickly; the dimensions and size estimate shown in the Export section help you check before exporting.

## Tips

- If the tool detects only one big frame, the sprites probably touch each other. Switch to **Manual grid** and enter the columns and rows.
- If the animation jitters, use **Manual grid**, change **Frame alignment**, or add per-frame **Margin** where needed.
- Use **Show split area** to confirm that the numbering and order of frames are correct before exporting.

## Under the hood

- Plain HTML, CSS, and JavaScript in one file.
- Frame detection uses row and column projections over a background mask.
- GIF encoder: median-cut color quantization, LZW compression, and per-frame local color tables.
- Animated WebP: frames are encoded by the browser and assembled into an animated container (RIFF `VP8X` / `ANIM` / `ANMF` chunks).
- Video: `MediaRecorder` on a canvas stream.
- Resampling: pica 10.0.3 (Lanczos3 with unsharp mask), with a canvas fallback.

## Browser support

| Feature | Chrome / Edge | Firefox | Safari |
| --- | --- | --- | --- |
| Detection, preview, GIF | Yes | Yes | Yes |
| Animated WebP export | Yes | Yes | No |
| Video export | WebM / MP4 | WebM | MP4 |

Support depends on browser version; treat the table as a guide.

## Project structure

```
.
├── sprite-animator.html   # the whole application
├── README.md
└── .gitignore
```

## Ideas for later

- Multi-row sprite sheets as separate directional animations (up, down, left, right).
- PNG sequence or ZIP export.
- Custom frame ordering and per-frame duration.
- Onion-skin preview.
