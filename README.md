# avi3d2wiggle

Convert Nintendo 3DS stereo AVI videos into wiggle WebM videos — entirely in the browser.

Inspired by [mpo2gif](https://github.com/pagarca/mpo2gif), which does the same for 3DS stereoscopic photos (MPO format).

## What it does

The Nintendo 3DS outer cameras record stereo video in AVI files containing **two MJPEG streams**:

| Stream | FourCC | Content |
|--------|--------|---------|
| 0 | `00dc` | Left eye · 480×240 · MJPEG · 20fps |
| 1 | `01wb` | Audio · ADPCM 16kHz |
| 2 | `02dc` | Right eye · 480×240 · MJPEG · 20fps |

This tool:
1. Parses the AVI RIFF structure in pure JS (no ffmpeg, no server)
2. Extracts both MJPEG streams frame by frame
3. Computes the stereo disparity via SAD block matching (auto) or user-selected focal point (manual)
4. Aligns both streams symmetrically around the focal depth
5. Exports a wiggle WebM video alternating left-eye / right-eye frames

The alternating L/R frames create the **wiggle stereoscopy** effect — a sense of 3D depth on a regular 2D screen.

## Usage

Just open `index.html` in a browser. No install, no dependencies, no server needed.

1. Drop your `.AVI` file from the 3DS
2. The tool auto-computes the stereo offset
3. Optionally click anywhere on the left frame to set a custom focal point
4. Adjust wiggle speed with the slider
5. Click **▶ PREVIEW** to see the live wiggle animation
6. Click **⬇ EXPORT WEBM** to record and download the output video

## Controls

| Control | Description |
|---------|-------------|
| Speed slider | Duration per wiggle frame (25–400ms) |
| Crop borders | Removes edge padding caused by the alignment shift |
| Auto focal | Uses SAD block matching on the center of the first frame |
| Click left image | Override the focal point manually at the clicked region |

## Technical notes

- **AVI parser**: Pure JS RIFF traverser — no ffmpeg.wasm dependency
- **Shift detection**: Sum of Absolute Differences (SAD) on a 50% center crop, horizontal-only search (stereo baseline is horizontal)
- **Alignment**: Symmetric shift — left frame shifts left by `halfDx`, right frame shifts right by `halfDx`. Objects at the focal depth appear stationary; others wiggle
- **Export**: `HTMLCanvasElement.captureStream()` + `MediaRecorder` → WebM (VP9 preferred, VP8 fallback)
- **Memory**: Raw JPEG frames are kept in memory; decoded `ImageBitmap`s are created per-frame and immediately closed

## Browser compatibility

Tested on Chrome 120+, Firefox 120+. Safari 17+ should work. The `captureStream()` export may not work in all contexts.

## Limitations

- Export speed is bounded by `frameMs × 2 × N` (real-time encoding via MediaRecorder)
- Shift is computed once from the first frame and applied globally (good for static scenes; for videos with large camera motion a per-frame shift would be more accurate)
- Audio is not included in the output (the wiggle video has no audio track)

## License

GPL-3.0 — same as mpo2gif.
