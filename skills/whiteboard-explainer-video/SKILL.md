---
name: whiteboard-explainer-video
description: Use when turning storyboard images, knowledge graphics, posters, or comic pages into a hand-drawn explainer video, especially for requests mentioning 手绘过程, 非文字手绘, content-driven scene timing, synchronized voiceover, Chinese captions, ChatCut timelines, or an exported MP4.
---

# Whiteboard Explainer Video

## Core rule

Let measured narration drive timing. Preserve all source text from the first frame. Animate only non-text illustrations, icons, arrows, frames, and decorations.

## Modes

| Mode | Scope |
| --- | --- |
| `full` | Take ordered images through an editable narrated result. |
| `render-only` | Produce only a silent, text-preserving hand-drawn base. |
| `post-only` | Take an existing base through voice and captions without rebuilding it. |

## Preflight

Read [input-output-contract.md](references/input-output-contract.md) before acting. Select the mode and ask only for a missing decision that would change execution.

Inspect ordered images, scripts, audio, and existing outputs. Create and record one dedicated temporary working directory.

## Workflow

1. In `full`, confirm the final script and voice before any paid TTS, then measure every page's generated audio. In `render-only`, require measured page audio or explicit page durations. In `post-only`, reuse the existing base and never rebuild it.
2. Read [timing-manifest.md](references/timing-manifest.md). Write the canonical `timing.json` from measured durations and required holds. Never truncate narration to satisfy a duration constraint.
3. Render with the resolved local whiteboard CLI, absolute paths, `--text-preserve`, and no `--draw-text` unless explicitly requested. Keep audio out of the render and concatenate the ordered silent scenes into `whiteboard_base.mp4`.
4. For authorized post-production, use the applicable ChatCut skills: **REQUIRED SUB-SKILL `chatcut:voice`** for voice generation; **REQUIRED SUB-SKILL `chatcut:chatcut-plugin-basics`** for timeline work; **REQUIRED SUB-SKILL `chatcut:transcription`** for captions; and **REQUIRED SUB-SKILL `chatcut:verification`** for inspection. Use **REQUIRED SUB-SKILL `chatcut:export`** only when export is explicit.
5. Place narration without overlaps or unintended gaps. Allow only post-narration holds documented in `timing.json`. Bind singleton captions only to the narration track, never image text or notes. Correct proper names as display text without changing approved spoken audio.
6. Complete [verification-checklist.md](references/verification-checklist.md). Delete every task-created temporary artifact that is not a deliverable, recheck temporary-directory size, and report created, deleted, retained, and intentionally retained items.

## Stop conditions

Stop before unconfirmed paid TTS. Stop for ambiguous image order, failed text protection, or an impossible duration constraint. Preserve an editable timeline and stop before export unless export was explicitly requested.

## Output

Report the selected mode, measured duration, and retained artifact paths routed by the contract, including `narration.md`, `timing.json`, and `whiteboard_base.mp4` where applicable. Report the ChatCut link or state, confirmed voice/audio and caption state, verification evidence, unverified items, and cleanup results.

Even for a plan-only response, explicitly state which task-created temporary artifacts will be deleted and which deliverables or editable state will be retained. Do not substitute a statement that no test temporary artifacts exist.

For plan-only or blocked `post-only`, state that later verification will independently compare page boundaries, narration start/end times, and caption cues, report alignment results, and list every unverified item.
