# Verification Checklist

## Preflight

- [ ] Confirm the selected mode and verify that all mode-specific required inputs are present.
- [ ] Confirm the source-image order and record the image-to-scene mapping.
- [ ] Create and record one explicit temporary working path before producing intermediate files.
- [ ] Confirm the preserve-text policy and the requested canvas and frame rate.
- [ ] For paid TTS, confirm the final script and voice before any generation.

## Timing/audio

- [ ] Measure actual audio files; do not time scenes from script estimates when audio exists.
- [ ] Verify scene indices and time ranges are ordered and contiguous, with no narration overlaps or unintended gaps; allow only post-narration holds documented in `timing.json`.
- [ ] Verify every narration end equals its scene start plus the measured audio duration.
- [ ] Check that coloring completes within 0.5 seconds of narration end when feasible.
- [ ] Listen across every scene boundary for clipped speech, accidental silence, duplicated audio, and abrupt transitions.
- [ ] Keep the post-narration hold within 0.4–1.0 seconds unless a documented constraint prevents it.
- [ ] Report duration conflicts; never truncate speech to meet a requested total.

## Visual

- [ ] Confirm all original source text is present and readable from the first frame of each scene.
- [ ] Confirm that non-text drawing and coloring visibly progress rather than appearing only as a static image.
- [ ] Confirm text is never redrawn, distorted, occluded, recolored, or revealed late.
- [ ] Inspect representative frames from the start, drawing phase, coloring phase, and end of every scene.
- [ ] Check transitions, framing, and the final hold for visual continuity.

## Captions/ChatCut

- [ ] Generate captions only from the narration track, not from image text or supplementary notes.
- [ ] Keep captions to a maximum of two lines and use natural semantic breaks.
- [ ] Apply display-only corrections for proper names and product names without altering the approved spoken audio.
- [ ] Check caption placement against important source text and visual focal points on representative frames.
- [ ] Verify caption timing against the actual narration and confirm that no sentence ending is clipped or flashed.
- [ ] Preserve an editable ChatCut timeline and record the confirmed voice/audio and caption state.

## Export

- [ ] Do not export by default; require an explicit export request.
- [ ] If export is requested, inspect the actual exported file rather than relying only on a timeline preview.
- [ ] Verify the requested container, codec, resolution, frame rate, audio presence, duration, captions, and first/last frames.
- [ ] Record the export path and the evidence used to inspect it.

## Cleanup report

- [ ] List every temporary file and directory created during the run.
- [ ] Delete all created intermediate files, caches, screenshots, dumps, logs, and temporary directories that are not final deliverables.
- [ ] Recheck temporary and export-directory sizes, especially for large audio, scene, render, and proxy files.
- [ ] List everything deleted.
- [ ] List final files and any intentionally retained temporary items, with a reason for each retained item.
- [ ] Report the selected mode, retained files, editable link, measured duration, voice/caption state, verification evidence, and anything unverified.
