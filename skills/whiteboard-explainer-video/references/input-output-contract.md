# Input and Output Contract

Choose one mode before creating media. Treat every “forbidden by default” item as outside the authorized scope unless the user explicitly requests it.

| Mode | Required inputs | Work allowed | Forbidden by default | Retained or returned state |
| --- | --- | --- | --- | --- |
| `full` | Ordered source images; either a per-page or full narration script, or explicit permission to draft it | Prepare narration, measure approved audio, render the text-preserving whiteboard base, and build captions and post-production state | Exporting a final video | `narration.md`, `timing.json`, `whiteboard_base.mp4`, and an editable ChatCut timeline |
| `render-only` | Ordered source images; either measured per-page audio or explicit per-page durations | Build timing data and render only the text-preserving whiteboard base | TTS, ChatCut work, captions, and export | `timing.json` and `whiteboard_base.mp4` |
| `post-only` | A base MP4, timing data or scene boundaries, narration text, and a confirmed voice or supplied audio | Synchronize the confirmed narration/audio and narration-derived captions in an editable timeline | Regenerating the whiteboard base and exporting a final video | The editable timeline plus its narration, voice/audio, and caption state |

## Defaults

- Use `full` unless the request explicitly limits the work to `render-only` or `post-only`.
- Preserve source-image text from the first frame; draw and color only non-text elements.
- Let measured audio duration drive scene timing whenever audio exists. Use explicit durations only when render-only work has no audio to measure.
- Generate original captions only from the narration track, never from source-image text, notes, or invented copy.
- Keep the timeline editable by default.

## Authorization boundaries

- Confirm both the final script and the selected voice before initiating paid TTS.
- Do not generate extra voice, render, caption, or export variants without a separate request.
- Do not export a final video without an explicit export request.
- Report any requested-duration conflict. Never truncate, accelerate, or otherwise compress speech unless the user explicitly relaxes the conflicting constraint.

## Completion report

Report all of the following, including explicit `none` or `unverified` values where applicable:

- Selected mode.
- Files created, updated, and retained.
- Editable project or timeline link, when available.
- Measured total duration.
- Confirmed voice/audio state and caption state.
- Verification evidence inspected.
- Anything not verified.
- Temporary files and directories created, deleted, and intentionally retained, with reasons for retention.
