# Timing Manifest

Use `timing.json` as the canonical, reconstructable scene schedule. The following valid JSON shows every required field; its values are illustrative, not default durations.

```json
{
  "schema_version": "1.0",
  "mode": "full",
  "fps": 30,
  "canvas": {
    "width": 1920,
    "height": 1080
  },
  "total_duration_seconds": 16.2,
  "duration_constraint": null,
  "scenes": [
    {
      "index": 1,
      "image": "images/001.png",
      "narration": "第一段已确认旁白。",
      "audio": "audio/001.wav",
      "audio_duration_seconds": 6.8,
      "text_policy": "preserve-from-first-frame",
      "start": 0.0,
      "draw_end": 4.2,
      "color_end": 6.8,
      "narration_end": 6.8,
      "end": 7.6
    },
    {
      "index": 2,
      "image": "images/002.png",
      "narration": "第二段已确认旁白。",
      "audio": "audio/002.wav",
      "audio_duration_seconds": 7.8,
      "text_policy": "preserve-from-first-frame",
      "start": 7.6,
      "draw_end": 11.8,
      "color_end": 15.4,
      "narration_end": 15.4,
      "end": 16.2
    }
  ]
}
```

`mode` is `full`, `render-only`, or `post-only`. `duration_constraint` is either `null` or the user-requested total duration in seconds; it records a constraint but does not override measured speech. The example above is the canonical fully populated `full`-mode form.

## Required and nullable scene fields

Include all canonical scene keys in every scene object, using `null` only where the selected mode permits it.

- **All modes:** Require `index`, `start`, `end`, and `text_policy`.
- **`full`:** Require `image`, `narration`, `audio`, `audio_duration_seconds`, `narration_end`, `draw_end`, and `color_end` after approved TTS exists. Do not render from estimated script duration.
- **`render-only`:** Require `image`, `draw_end`, and `color_end`. With measured page audio, also require `audio`, `audio_duration_seconds`, and `narration_end`; `narration` may be the supplied text or `null`. With explicit page durations and no audio, `narration`, `audio`, `audio_duration_seconds`, and `narration_end` may all be `null`.
- **`post-only`:** Require `narration`. When only an existing base video and page boundaries are known, `image`, `draw_end`, and `color_end` may be `null`. Once a voice is generated or existing audio is reused, require `audio`, `audio_duration_seconds`, and `narration_end`.

## Invariants

- Store scenes in source page order with one-based, contiguous `index` values. When `image` is `null` in `post-only`, use the reconstructed page-boundary order. Scene ranges are contiguous: the first `start` is `0`, and every later `start` equals the preceding `end`.
- For every scene with audio, set `narration_end = start + audio_duration_seconds`, using the measured audio duration rather than an estimate.
- When render phase fields are present, enforce `start <= draw_end <= color_end <= end`.
- For all non-null phase ends, enforce `scene.end >= max(scene.color_end, scene.narration_end)`, equivalently `end_seconds >= max(color_end_seconds, narration_end_seconds)`. A scene may never end before its narration.
- When feasible, keep color synchronized to narration within ±0.5 seconds: `abs(color_end - narration_end) <= 0.5`.
- For narrated scenes, define the post-narration hold as `end - narration_end`; record this intentional silence between narration clips in `timing.json` and keep it within 0.4–1.0 seconds unless an explicitly documented transition constraint makes that infeasible. Forbid any other narration gap.
- For rendered scenes, define the visual hold as `end - color_end` and require it to be non-negative.
- Set `total_duration_seconds` equal to the final scene’s `end`. It is content-derived and never fixed at 150 seconds.
- Never satisfy `duration_constraint` by truncating speech. If measured narration plus required holds exceeds the constraint, report the conflict and wait for the user to relax a requirement.

## Phase-to-execution mapping

The whiteboard renderer has drawing and tail-color phases but no final hold. For a scene with render phase fields, derive:

```text
start_seconds = scene.start
draw_end_seconds = scene.draw_end
color_end_seconds = scene.color_end
narration_end_seconds = scene.narration_end
end_seconds = scene.end
fps = timing.fps
canvas_width = timing.canvas.width
canvas_height = timing.canvas.height
render_duration = color_end_seconds - start_seconds
color_duration = color_end_seconds - draw_end_seconds
hold_duration = end_seconds - color_end_seconds
post_narration_hold_duration = end_seconds - narration_end_seconds
```

Compute a derived duration only when its source fields are non-null. In particular, compute `post_narration_hold_duration` only when `narration_end_seconds` exists.

Passing `render_duration` as `--duration` makes the renderer allocate drawing as `render_duration - color_duration` and coloring as `color_duration`. Passing the whole scene duration would incorrectly consume the intended hold inside the renderer. Append a cloned final-frame hold after rendering whenever `hold_duration > 0` to produce the final scene clip.

Narration is placed separately from the silent whiteboard base. `narration_end` is an alignment and verification field; it does not control the renderer or replace the measured audio duration.

## Engine resolution and commands

Resolve the engine in this order:

1. Use a user-provided or configured absolute engine path.
2. Otherwise, locate a project- or workspace-local `whiteboard-video-engine` and resolve it to an absolute path.
3. If neither exists, stop and ask the user for the engine location. Never download the engine or any model silently.

Use absolute paths for the engine, source image, intermediate render, and final scene output. For each rendered scene:

```sh
ENGINE_ROOT=/absolute/path/to/whiteboard-video-engine
IMAGE_PATH=/absolute/path/to/source/001.png
RENDERED_SCENE_PATH=/absolute/path/to/work/scene-001-rendered.mp4
FINAL_SCENE_PATH=/absolute/path/to/work/scene-001.mp4

"$ENGINE_ROOT/.venv/bin/whiteboard" render-photo "$IMAGE_PATH" \
  -o "$RENDERED_SCENE_PATH" \
  --duration "$render_duration" \
  --fps "$fps" \
  --width "$canvas_width" \
  --height "$canvas_height" \
  --tail-color "$color_duration" \
  --text-preserve

if awk -v hold="$hold_duration" 'BEGIN { exit !(hold > 0) }'; then
  ffmpeg -y -i "$RENDERED_SCENE_PATH" \
    -vf "tpad=stop_mode=clone:stop_duration=$hold_duration" \
    -c:v libx264 -pix_fmt yuv420p -an "$FINAL_SCENE_PATH"
else
  cp -- "$RENDERED_SCENE_PATH" "$FINAL_SCENE_PATH"
fi
```

The conditional runs the `ffmpeg` hold step only when `hold_duration > 0`; otherwise the rendered clip becomes the final scene clip without re-encoding. Keep narration/audio out of this base-render command and add it in the authorized post-production stage.
