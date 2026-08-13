---
name: video-depth-map
description: Use when a video must be converted into a grayscale depth-map video, depth-conditioned control clip, or structure guide for dance/video replacement, Wan, MiniMax H3, ControlNet, or another video-generation workflow.
---

# Video Depth Map

## Scope

Generate or prepare a time-aligned grayscale depth video from an authorized source. Treat it as a structural control asset, not a final black-and-white grade. Preserve the source video and write the depth result beside the task outputs.

## Route and current machine state

Always enter through `$media-studio-orchestrator`, then route here for the depth stage. This Skill does not start ComfyUI for analysis-only work. For generation, reuse `127.0.0.1:8188` or use the user's visible Desktop according to the global ComfyUI rules.

Current status on this workstation is `partial`: FFmpeg/FFprobe and the video-learning Python environment are available, but no verified depth-estimation model or depth custom node is installed. `D:\Comfy-Desktop\ComfyUI-Shared\models\geometry_estimation` is empty; do not claim local depth generation until a model/node is installed and an output is rendered.

## Research-backed options

The recommended model family for this task is [Video-Depth-Anything](https://github.com/DepthAnything/Video-Depth-Anything), an Apache-2.0 codebase designed for consistent depth on very long videos. Its official README lists relative-depth checkpoints of approximately 28.4M, 113.1M, and 381.8M parameters (Small/Base/Large), with FP16 GPU measurements on A100 of about 6.8 GB, 14 GB, and 23.6 GB respectively. Current model files are approximately 116 MB for Small and 458 MB for Base; verify the Large file size from its current download response before installation. The Small checkpoint is Apache-2.0; the Base and Large checkpoints are listed as CC-BY-NC-4.0, so check the license before commercial use.

For ComfyUI, [ComfyUI-Video-Depth-Anything](https://github.com/yuvraj108c/ComfyUI-Video-Depth-Anything) provides an unofficial node and example workflow; it downloads models under `ComfyUI/models/videodepthanything` and supports Small/Base/Large plus metric variants. [ComfyUI-Depth-Anything-Tensorrt](https://github.com/yuvraj108c/ComfyUI-Depth-Anything-Tensorrt) provides TensorRT image depth nodes and a temporal stabilizer, but its README describes a separate engine-export/model setup and does not by itself prove the long-video Video-Depth-Anything route. [ComfyUI-DepthAnythingV2](https://github.com/kijai/ComfyUI-DepthAnythingV2) is a popular single-image/frame node; use it for quick previews, not as the production temporal route.

The practical choices are:

| Choice | Use | Cost/limitation |
|---|---|---|
| Official Video-Depth-Anything CLI | Best first local route for stable depth video | Separate Python environment and checkpoint; no ComfyUI graph by itself |
| ComfyUI-Video-Depth-Anything | Best if the result must feed a visible ComfyUI workflow | Custom node plus dependencies/models; current repository was tested by its author on a newer RTX/CUDA stack, so run a small smoke before production |
| Depth Anything V2/TensorRT | Fast still/frame preview or existing ComfyUI image workflow | Temporal stability is weaker unless an additional stabilizer is used; not equivalent to VDA |
| Web depth-map service | Quick experiment without local installation | Upload/privacy, quota, unknown model/parameters, and no local reproducibility |

For this RTX 4090, start with Video-Depth-Anything Small or Base in an isolated D-drive environment. Do not install into the video-learning venv or ComfyUI's shared Python until compatibility is checked. Small is the conservative first smoke; Base is the quality comparison; Large is not the first choice because its published FP16 memory figure is close to a 24 GB card's practical ceiling once video buffers and other applications are included.

The existing video-learning `faster_whisper` package and D-drive Tesseract are transcription/OCR dependencies only. They are not depth-map dependencies and do not need to be installed for this Skill.

## Choose an execution path

| Situation | Route |
|---|---|
| User asks only what a tutorial means | `video-learning`; inspect frames/OCR/audio as available; do not generate depth video |
| User asks to generate a local depth video and a verified depth model exists | Use the installed local depth runtime; process every frame with the source FPS and duration preserved |
| A ComfyUI depth/video node is installed and its model is visible in `/object_info` | Use `comfyui-video-workflow-author`; save matching Canvas JSON and API JSON before submission |
| No local depth model/node | State `partial`; offer installation with target D-drive path and official link, or let the user use a web service and provide the exported depth video |

Do not silently download a large model. Before installation, report the model, approximate size, target path, runtime/Python/CUDA requirements, and whether it shares or isolates the existing ComfyUI environment.

## Depth-video contract

1. Probe source streams with FFprobe. Record width, height, FPS, duration, codec, rotation, and audio presence.
2. Split at hard scene cuts when detected; depth normalization may reset per shot. Keep source timecodes.
3. Run the depth estimator frame by frame or in bounded batches. Preserve frame count, FPS, duration, aspect ratio, and audio separately. Never overwrite the source.
4. Write a grayscale preview and an encoded depth video. Record model, version/hash, input/output resolution, normalization, near/far polarity, FPS, codec, and elapsed time.
5. Review entry/peak/exit frames and a short playable output. Check subject silhouette, hands/feet, hair, thin garments, occlusion order, background separation, temporal flicker, and shot-boundary resets.
6. Mark the result `pass` only when the requested structural properties are visibly retained; otherwise use `partial` or `blocked` with the exact defect.

The usual interpretation is bright-near/dark-far, but some tools invert it. Confirm polarity from the tool's documentation or a known scene before sending the clip to ControlNet.

## Using the depth clip for video replacement

Use the depth clip as a control/conditioning input, then provide a separately authorized character reference or role card. Depth preserves structure; it does not guarantee identity, facial detail, clothing texture, dialogue, lip sync, or audio. For Wan or MiniMax H3, run a low-resolution short smoke first, then the target specification. Keep depth-quality acceptance separate from generated-video identity and production-quality acceptance.

When the requested object or garment is missing from a structural guide, first search the authorized local asset library and extract a suitable guide; if none exists, offer image generation to create a guide; ask the user only after both routes fail. Do not invent that a missing guide was present.

## Evidence and delivery

For every task, report the selected route and state (`ready`, `partial`, `missing`, or `blocked`), checks actually run, and remaining acceptance boundary. Use clickable links for the depth video, preview, report, JSON, and representative frames. A tutorial claim or web result is not local acceptance.

This Skill does not create subtitles or translations unless explicitly requested by the user; if requested, hand the subtitle stage to `video-learning` and retain its transcription quality gate.

## Common failure modes

- **Looks like an ordinary black-and-white copy:** the output is a color-grade, not a depth map. Re-run the estimator and inspect depth ordering.
- **Broken hair/hands/feet:** mark partial; try higher input resolution or a model suited to human/portrait edges.
- **Flickering gray levels:** normalize consistently over a shot or use a temporally stable estimator; never hide the defect with a final quality label.
- **Model/node installed but dropdown empty:** fix ComfyUI model paths and verify `/object_info` before submission.
- **Good depth but poor replacement:** separate the depth result from the generator's identity, prompt, or temporal failure.
