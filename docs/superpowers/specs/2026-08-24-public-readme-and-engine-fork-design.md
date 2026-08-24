# Public README and Engine Fork Design

## Goal

Turn `2510466299/whiteboard-explainer-video-skill` into a public, understandable, and installable Codex Skill repository while preserving clear attribution to [`gnipbao/whiteboard-video-engine`](https://github.com/gnipbao/whiteboard-video-engine).

The public release uses two repositories with separate responsibilities:

- `2510466299/whiteboard-explainer-video-skill`: orchestration instructions, timing contracts, installation guidance, and user-facing documentation.
- `2510466299/whiteboard-video-engine`: a GitHub fork that preserves upstream history and contains the compatible text-preservation and Apple-device changes required by the Skill.

## Repository relationship

The README must state that the Skill is an independent orchestration layer built around the MIT-licensed upstream engine. It must link the original project and its license, identify the compatible fork, summarize the modifications, and avoid implying endorsement by the upstream author.

The engine fork must retain the upstream commit history and existing MIT `LICENSE`. Model weights, generated media, virtual environments, caches, and third-party assets without verified redistribution rights must not be committed.

## Skill repository contents

Add these public-facing files:

- `README.md`: primary bilingual-friendly Chinese documentation with English technical names and commands.
- `LICENSE`: MIT license for the Skill repository, with copyright attributed to GitHub account `2510466299`.

Keep the existing Skill package and design documents. Do not copy the engine source into this repository.

## README structure

The README will contain:

1. Project summary and scope.
2. A Mermaid diagram showing images/scripts/audio flowing through the engine, timing manifest, and optional ChatCut post-production.
3. Feature list: non-text drawing, source-text preservation, narration-driven timing, CUDA to MPS to CPU device selection, three operating modes, captions, and editable timeline handoff.
4. Upstream attribution and a concise list of fork-specific changes.
5. Requirements split into core rendering and optional line-art/ChatCut capabilities.
6. Installation steps for the engine fork and the Codex Skill, using Python 3.12, `uv`, and FFmpeg.
7. Quick-start prompts for `full`, `render-only`, and `post-only`.
8. Input/output contracts and retained artifacts.
9. Authorization boundaries for paid TTS and export.
10. Troubleshooting for MPS availability, missing models, text preservation, and duration conflicts.
11. Limitations, privacy, license, and acknowledgements.

Commands must be verified against the actual fork before publication. The README must distinguish required dependencies from optional services and must not claim that model weights are bundled.

## Engine fork publication

Create a real GitHub fork of `gnipbao/whiteboard-video-engine` under account `2510466299`. Synchronize it with the current upstream `main`, then apply the two local commits that add the release design and text-preserving/device-aware rendering. If Git history already applies cleanly, push the existing commits; otherwise cherry-pick them onto the current upstream without rewriting upstream history.

Add a short fork notice near the top of the fork README that links back to the original project and the Skill repository. Do not replace the upstream README or license.

## Visibility and release order

1. Create and verify the public engine fork.
2. Update and validate the Skill README against that fork.
3. Push the Skill repository changes while it is still private.
4. Verify the remote file tree, links, commands, and license.
5. Change the Skill repository visibility to public only after the checks pass.

This order avoids publishing documentation that points to a missing or incompatible engine.

## Verification

Before declaring completion:

- Run the official Skill validator.
- Run the engine test suite relevant to device selection, CLI flags, and text preservation.
- Verify README commands in a clean temporary checkout without downloading model weights or calling paid services.
- Confirm both GitHub repositories are public and the engine repository is recorded by GitHub as a fork of `gnipbao/whiteboard-video-engine`.
- Confirm both repositories expose the required MIT license and attribution.
- Scan the committed trees for credentials, model weights, generated media, virtual environments, caches, and oversized files.
- Check every README link and report any runtime behavior that remains unverified.

## Temporary-file policy

Use one recorded directory under `/private/tmp` for clones, validation caches, and command logs. Remove that directory after the remote state is verified. Retain only the GitHub repositories and the existing local project sources because they are final deliverables or source-of-truth material.
