# Whiteboard Explainer Video Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and verify a project-level `whiteboard-explainer-video` Skill that orchestrates content-driven whiteboard rendering, narration timing, ChatCut voiceover/captions, and cleanup without assuming a fixed 150-second duration.

**Architecture:** Keep the main `SKILL.md` as a concise router for `full`, `render-only`, and `post-only` modes. Put the detailed input/output contract, `timing.json` rules, and verification/cleanup gates in three one-level reference files. Reuse the existing `whiteboard-video-engine` and ChatCut skills instead of copying binaries, models, or media.

**Tech Stack:** Markdown Skill package, YAML UI metadata, `whiteboard-video-engine` CLI, ChatCut skills/MCP, Python skill validation through `uv` + PyYAML, Git.

---

## File map

- Create: `skills/whiteboard-explainer-video/SKILL.md` — trigger metadata, mode routing, core workflow, authorization boundaries, and reference navigation.
- Create: `skills/whiteboard-explainer-video/agents/openai.yaml` — UI name, short description, and an explicit `$whiteboard-explainer-video` starter prompt.
- Create: `skills/whiteboard-explainer-video/references/input-output-contract.md` — required/optional inputs and retained outputs for each mode.
- Create: `skills/whiteboard-explainer-video/references/timing-manifest.md` — canonical `timing.json` example and timing invariants.
- Create: `skills/whiteboard-explainer-video/references/verification-checklist.md` — preflight, audio, visual, caption, export, and cleanup checks.
- Temporary only: `/private/tmp/whiteboard-explainer-video-skill-20260716/` — baseline/forward-test notes and the `init_skill.py` scaffold; delete before completion.

No production source file under `whiteboard-video-engine/` is modified in this plan.

### Task 1: RED — establish behavior failures before the Skill exists

**Files:**
- Create temporarily: `/private/tmp/whiteboard-explainer-video-skill-20260716/baseline.md`
- Inspect: `docs/superpowers/specs/2026-07-16-whiteboard-explainer-video-skill-design.md`

- [ ] **Step 1: Record the temporary test directory**

Run:

```bash
mkdir -p /private/tmp/whiteboard-explainer-video-skill-20260716
```

Expected: the directory exists only for this task and is listed in the final cleanup report.

- [ ] **Step 2: Run four fresh baseline scenarios without loading the new Skill**

Use independent agents with no access to the intended Skill text. Do not allow filesystem writes. Use these prompts verbatim:

```text
FULL: 目录里有 10 张中文知识图和逐页口播稿。请说明你会如何生成完整手绘讲解视频。音色用 dayi，时长按内容决定，只手绘非文字部分，添加中文字幕，先不要导出。

RENDER-ONLY: 我已经有逐页旁白音频和 8 张中文海报，只生成非文字手绘底片，不要生成新配音、不要进入 ChatCut。请给出执行方案和交付物。

POST-ONLY: 我已经有 whiteboard_base.mp4、timing.json 和逐页口播稿。请在 ChatCut 里同步大壹旁白和中文字幕，修正 Claude、Windsurf 等专有名词，先不要导出。

CONFLICT: 成片必须限制在 60 秒，但现成旁白音频是 80 秒，不允许删稿。请直接处理并说明你会怎么做。
```

For each response, score these observable behaviors:

```text
[ ] selects the correct mode
[ ] treats real audio duration as authoritative
[ ] preserves source text by default
[ ] does not create forbidden TTS/ChatCut/export work in partial modes
[ ] names narration.md, timing.json, whiteboard_base.mp4, or the relevant subset
[ ] refuses to truncate or unreadably accelerate the 80-second narration
[ ] includes verification and temporary-file cleanup
```

- [ ] **Step 3: Verify RED**

Expected: at least one baseline response misses at least one required behavior. If all four responses pass every item, strengthen only the ambiguous prompt until a real failure is observed; do not write the Skill before a discriminating failure exists.

- [ ] **Step 4: Record exact baseline evidence**

Write `/private/tmp/whiteboard-explainer-video-skill-20260716/baseline.md` with this fixed structure:

```markdown
# Baseline

## Full
- Missing behavior:
- Exact evidence:

## Render only
- Missing behavior:
- Exact evidence:

## Post only
- Missing behavior:
- Exact evidence:

## Duration conflict
- Missing behavior:
- Exact evidence:
```

Do not turn this file into a retained project document; it is test evidence and must be deleted in Task 6.

### Task 2: Generate a clean scaffold and create the three contracts

**Files:**
- Create temporarily: `/private/tmp/whiteboard-explainer-video-skill-20260716/scaffold/whiteboard-explainer-video/`
- Create: `skills/whiteboard-explainer-video/references/input-output-contract.md`
- Create: `skills/whiteboard-explainer-video/references/timing-manifest.md`
- Create: `skills/whiteboard-explainer-video/references/verification-checklist.md`

- [ ] **Step 1: Run the official initializer in the recorded temporary directory**

Run:

```bash
python3 /Users/undersdong/.codex/skills/.system/skill-creator/scripts/init_skill.py \
  whiteboard-explainer-video \
  --path /private/tmp/whiteboard-explainer-video-skill-20260716/scaffold \
  --resources references \
  --interface display_name='Whiteboard Explainer Video' \
  --interface short_description='Build narration-timed hand-drawn explainer videos.' \
  --interface default_prompt='Use $whiteboard-explainer-video to turn these storyboard images into a narration-timed hand-drawn explainer video while preserving image text.'
```

Expected output contains:

```text
[OK] Created skill directory
[OK] Created SKILL.md
[OK] Created agents/openai.yaml
[OK] Created references/
```

Inspect the scaffold, but create final project files with `apply_patch` rather than copying generated placeholders.

- [ ] **Step 2: Write `input-output-contract.md`**

Create a compact reference containing this exact contract:

```markdown
# Input and output contract

## Modes

| Mode | Required input | Forbidden work by default | Retained output |
| --- | --- | --- | --- |
| `full` | ordered images plus per-page script, full script with split permission, or permission to draft | export without explicit request | `narration.md`, `timing.json`, `whiteboard_base.mp4`, editable ChatCut timeline |
| `render-only` | ordered images plus measured per-page audio or explicit scene durations | new TTS, ChatCut editing, captions, export | `timing.json`, `whiteboard_base.mp4` |
| `post-only` | `whiteboard_base.mp4`, timing data or reconstructable boundaries, narration, and confirmed voice/audio | regenerating the base video; export without explicit request | editable ChatCut timeline and caption state |

## Defaults

- Choose `full` unless the request clearly limits the work.
- Use `preserve-text`; never run source Chinese through line-art reconstruction unless explicitly requested.
- Derive scene length from measured audio. Treat a user total/min/max duration as a constraint, not a default.
- Add original-language captions in `full`; bind them only to the narration track.
- Stop at an editable ChatCut timeline unless export is explicit.

## Authorization

Confirm script and voice before paid TTS. Do not create extra variants. Do not submit an export unless the request covers export. Report duration conflicts instead of truncating speech.

## Completion report

Return mode, retained files, ChatCut project/timeline link when applicable, measured duration, voice and caption state, verification evidence, unverified items, and created/deleted/retained temporary-file lists.
```

- [ ] **Step 3: Write `timing-manifest.md`**

Include this canonical example:

```json
{
  "schema_version": "1.0",
  "mode": "full",
  "fps": 30,
  "canvas": {"width": 1080, "height": 1440},
  "total_duration_seconds": 18.6,
  "duration_constraint": {"target": null, "min": null, "max": null},
  "scenes": [
    {
      "index": 1,
      "image": "01.png",
      "narration": "第一页旁白。",
      "audio": "audio/scene_01.mp3",
      "audio_duration_seconds": 17.8,
      "text_policy": "preserve-text",
      "start_seconds": 0.0,
      "draw_end_seconds": 10.2,
      "color_end_seconds": 17.6,
      "narration_end_seconds": 17.8,
      "end_seconds": 18.6
    }
  ]
}
```

Define these invariants below the example:

```text
- Scene indexes follow image order and start at 1.
- First scene starts at 0; each later start equals the prior end.
- narration_end_seconds = start_seconds + measured audio duration.
- draw_end_seconds <= color_end_seconds <= end_seconds.
- abs(color_end_seconds - narration_end_seconds) <= 0.5 when visually feasible.
- end_seconds - narration_end_seconds is normally 0.4–1.0 seconds.
- total_duration_seconds equals the final scene end; it is never fixed at 150.
- A duration constraint may change speed or holds but may not truncate speech.
```

Also include the engine command pattern, explicitly retaining measured duration:

```bash
whiteboard-video-engine/.venv/bin/whiteboard render-photo <image> \
  -o <scene.mp4> \
  --duration <scene-end-minus-scene-start> \
  --fps <fps> \
  --text-preserve
```

- [ ] **Step 4: Write `verification-checklist.md`**

Use these sections and checks:

```markdown
# Verification and cleanup

## Preflight
- Image count/order is explicit; dimensions and readable text are checked.
- Mode and required inputs are complete.
- Temporary directory path is recorded.

## Timing and audio
- Actual audio duration, not character count, drives final timing.
- Every narration item is contiguous with the intended page.
- Coloring ends within 0.5 seconds of narration when feasible.
- No accidental silence appears at scene boundaries.

## Visual
- Original text is legible from the first visible frame.
- Non-text elements visibly draw and color over time.
- Representative start, middle, page-boundary, and end frames are inspected.

## Captions and ChatCut
- Captions source only the narration track.
- Chinese uses at most two lines with readable outline/background.
- Names such as Claude, Gemini, Windsurf, Token, and API Key are corrected as display text.
- Caption placement is checked against important source text.

## Export
- No export exists unless explicitly requested.
- If requested, the render reaches completion and the actual file is inspected.

## Cleanup report
- Delete preview frames, screenshots, caches, temporary audio slices, logs, and test exports.
- Retain only source material, final narration, timing manifest, base video, requested export, and editable project assets.
- Report created, deleted, and retained paths with reasons.
```

- [ ] **Step 5: Check references before writing the main Skill**

Run:

```bash
rg -n '150|preserve-text|render-only|post-only|export|temporary|字幕|旁白' \
  skills/whiteboard-explainer-video/references
```

Expected: `150` appears only in a sentence that says timing is not fixed at 150; all other required keywords have matches.

- [ ] **Step 6: Commit the contracts**

```bash
git add skills/whiteboard-explainer-video/references
git commit -m "docs: define whiteboard explainer skill contracts"
```

Expected: only the three reference files are included.

### Task 3: GREEN — write the minimal router Skill and UI metadata

**Files:**
- Create: `skills/whiteboard-explainer-video/SKILL.md`
- Create: `skills/whiteboard-explainer-video/agents/openai.yaml`

- [ ] **Step 1: Write `SKILL.md` with trigger-only frontmatter**

Use this frontmatter exactly:

```yaml
---
name: whiteboard-explainer-video
description: Use when turning storyboard images, knowledge graphics, posters, or comic pages into a hand-drawn explainer video, especially for requests mentioning 手绘过程, 非文字手绘, content-driven scene timing, synchronized voiceover, Chinese captions, ChatCut timelines, or an exported MP4.
---
```

Write the body in imperative language with these concise sections:

```markdown
# Whiteboard Explainer Video

## Core rule

Drive timing from measured narration audio. Preserve source text by default; animate only non-text illustration, icons, arrows, frames, and decoration.

## Choose a mode

| Request | Mode |
| --- | --- |
| Images through editable narrated result | `full` |
| Silent hand-drawn base only | `render-only` |
| Existing base through voiceover/captions | `post-only` |

Read `references/input-output-contract.md` before taking action. If inputs are incomplete, ask only for the missing decision that changes execution.

## Execute

1. Inspect ordered images, script/audio, existing outputs, and project state. Record a dedicated temporary directory.
2. In `full`, confirm the final script and concrete voice before paid TTS. Generate or reuse per-page audio, then measure it. In `render-only`, require measured audio or explicit durations. In `post-only`, do not rebuild the base.
3. Read `references/timing-manifest.md`; write `timing.json` from measured audio and visual complexity. Treat total duration as an optional constraint. Never truncate narration to satisfy it.
4. Render each image with the local `whiteboard` CLI and `--text-preserve`. Do not pass `--draw-text` unless explicitly requested. Concatenate scenes into `whiteboard_base.mp4`.
5. For ChatCut work, use the applicable ChatCut skills. **REQUIRED SUB-SKILL:** Use `chatcut:voice` before TTS, `chatcut:chatcut-plugin-basics` for timeline edits, `chatcut:transcription` for transcript readiness, and `chatcut:verification` before completion. Use `chatcut:export` only when export is explicit.
6. Keep narration contiguous with page boundaries. Bind the singleton captions overlay only to the narration track; correct names as display text without changing audio.
7. Read and complete `references/verification-checklist.md`. Delete all task-created temporary artifacts and report created, deleted, and retained paths.

## Stop conditions

- Stop before paid TTS when script or voice is unconfirmed.
- Stop when image order is ambiguous or text protection fails on a representative page.
- Report an impossible duration constraint; do not hide it with truncation or unreadable speed.
- Stop at the editable timeline unless the user explicitly requested export.

## Output

Return the selected mode, measured duration, retained artifact paths, ChatCut link/state when applicable, verification evidence, unverified items, and cleanup report.
```

- [ ] **Step 2: Generate `agents/openai.yaml` deterministically**

Run in the temporary scaffold first:

```bash
python3 /Users/undersdong/.codex/skills/.system/skill-creator/scripts/generate_openai_yaml.py \
  /private/tmp/whiteboard-explainer-video-skill-20260716/scaffold/whiteboard-explainer-video \
  --interface display_name='Whiteboard Explainer Video' \
  --interface short_description='Build narration-timed hand-drawn explainer videos.' \
  --interface default_prompt='Use $whiteboard-explainer-video to turn these storyboard images into a narration-timed hand-drawn explainer video while preserving image text.'
```

Then create the final project file with `apply_patch` containing:

```yaml
interface:
  display_name: "Whiteboard Explainer Video"
  short_description: "Build narration-timed hand-drawn explainer videos."
  default_prompt: "Use $whiteboard-explainer-video to turn these storyboard images into a narration-timed hand-drawn explainer video while preserving image text."
```

- [ ] **Step 3: Run static checks**

Run:

```bash
wc -l -w skills/whiteboard-explainer-video/SKILL.md
rg -n '[T]ODO|[T]BD|[F]IXME|@skills|150 seconds|150 秒默认' skills/whiteboard-explainer-video
git diff --check -- skills/whiteboard-explainer-video
```

Expected:

- `SKILL.md` is under 500 lines and roughly under 500 English-delimited words;
- placeholder scan returns no matches;
- `git diff --check` is silent.

- [ ] **Step 4: Run the official validator**

Run:

```bash
uv run --with pyyaml python \
  /Users/undersdong/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/whiteboard-explainer-video
```

Expected:

```text
Skill is valid!
```

- [ ] **Step 5: Commit the main Skill**

```bash
git add skills/whiteboard-explainer-video/SKILL.md skills/whiteboard-explainer-video/agents/openai.yaml
git commit -m "feat: add whiteboard explainer video skill"
```

Expected: only `SKILL.md` and `agents/openai.yaml` are included.

### Task 4: GREEN — repeat the same scenarios with the Skill loaded

**Files:**
- Read: `skills/whiteboard-explainer-video/SKILL.md`
- Read as routed: the three files in `skills/whiteboard-explainer-video/references/`
- Update temporarily: `/private/tmp/whiteboard-explainer-video-skill-20260716/forward-test.md`

- [ ] **Step 1: Run the four Task 1 prompts through fresh agents with the Skill**

Use the same prompts verbatim. Instruct each fresh agent only:

```text
Use $whiteboard-explainer-video at skills/whiteboard-explainer-video to answer the request. Do not modify files or call paid/external generation tools; provide the intended execution and output contract.
```

Do not provide the expected answers or baseline diagnosis.

- [ ] **Step 2: Verify GREEN**

Every scenario must pass all applicable Task 1 observable behaviors. In particular:

```text
FULL -> full mode; measured TTS drives timing; preserve text; no export.
RENDER-ONLY -> no TTS and no ChatCut; timing.json + base video only.
POST-ONLY -> no base regeneration; captions source narration only; no export.
CONFLICT -> report the unsatisfied 60-second constraint; do not truncate 80-second audio.
```

If a response fails, update only the instruction/reference that addresses the observed gap, then rerun that scenario and one adjacent mode to catch regressions.

- [ ] **Step 3: Record raw results**

Write `forward-test.md` with pass/fail per observable behavior and exact failure text if any. Keep this temporary until the final audit, then delete it.

### Task 5: REFACTOR — close observed gaps and verify the package

**Files:**
- Modify if required: `skills/whiteboard-explainer-video/SKILL.md`
- Modify if required: `skills/whiteboard-explainer-video/references/*.md`
- Modify if required: `skills/whiteboard-explainer-video/agents/openai.yaml`

- [ ] **Step 1: Fix only evidence-backed gaps**

Do not add speculative sections. Typical allowed fixes are:

```text
- clarify mode routing when an existing base video is present;
- make paid TTS confirmation explicit;
- strengthen narration-track-only caption sourcing;
- strengthen duration-conflict refusal;
- remove duplicated instructions from SKILL.md into a reference.
```

- [ ] **Step 2: Re-run focused scenarios**

Expected: the failed scenario and one neighboring mode both pass after the edit.

- [ ] **Step 3: Re-run all static validation**

```bash
uv run --with pyyaml python \
  /Users/undersdong/.codex/skills/.system/skill-creator/scripts/quick_validate.py \
  skills/whiteboard-explainer-video
rg -n '[T]ODO|[T]BD|[F]IXME|150 seconds|150 秒默认' skills/whiteboard-explainer-video
git diff --check -- skills/whiteboard-explainer-video
```

Expected: validator says `Skill is valid!`; scans and diff check are silent.

- [ ] **Step 4: Confirm UI metadata matches the Skill**

Check:

```text
- display_name names the Skill, not one mode.
- short_description is 25–64 characters.
- default_prompt contains the literal $whiteboard-explainer-video.
- no icon, brand color, or MCP dependency is declared without user input.
```

- [ ] **Step 5: Commit evidence-backed refinements**

```bash
git add skills/whiteboard-explainer-video
git commit -m "test: harden whiteboard explainer skill workflow"
```

Skip this commit if no file changed.

### Task 6: Completion audit, cleanup, and handoff

**Files:**
- Verify: `skills/whiteboard-explainer-video/**`
- Delete: `/private/tmp/whiteboard-explainer-video-skill-20260716/`

- [ ] **Step 1: Audit every design acceptance criterion**

Build an evidence table in the working notes with one row for each of the 11 criteria in the design spec. Evidence must point to a concrete Skill line, reference line, validator output, or forward-test result. Treat missing evidence as incomplete work.

- [ ] **Step 2: Verify the committed file set**

Run:

```bash
git status --short -- skills/whiteboard-explainer-video
git log --oneline --decorate -5 -- skills/whiteboard-explainer-video
find skills/whiteboard-explainer-video -maxdepth 3 -type f -print | sort
```

Expected: exactly five retained files: `SKILL.md`, `agents/openai.yaml`, and three references. No README, model, media, cache, test output, or placeholder remains.

- [ ] **Step 3: Delete all task-created temporary material**

Delete only the recorded directory:

```bash
rm -rf /private/tmp/whiteboard-explainer-video-skill-20260716
```

Then verify:

```bash
test ! -e /private/tmp/whiteboard-explainer-video-skill-20260716
find /private/tmp -maxdepth 1 -name 'whiteboard-explainer-video-skill-*' -print
```

Expected: both commands produce no retained task directory or file.

- [ ] **Step 4: Report the outcome**

Return:

```text
- created Skill files and purpose;
- RED and GREEN scenario results;
- official validator and static-check results;
- created/deleted/retained temporary files;
- no model downloads, no paid TTS, and no final video export during validation;
- remaining unverified items, if any;
- the project-local Skill invocation: $whiteboard-explainer-video.
```
