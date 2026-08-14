---
name: video-depth-map
description: Use when a video must be converted into a grayscale depth-map video, depth-conditioned control clip, or structure guide for dance/video replacement, Wan, MiniMax H3, ControlNet, or another video-generation workflow.
---

# Video Depth Map

## Scope

Generate or prepare a time-aligned grayscale depth video from an authorized source. Treat it as a structural control asset, not a final black-and-white grade. Preserve the source video and write the depth result beside the task outputs.

## Output ownership

If the depth map is a stage of a short-drama or video-production project, keep the final grayscale video, optional color preview, metadata, and acceptance record with that project. Otherwise use `VIDEO_DEPTH_MAP_ROOT\<job>\` (default `D:\MediaStudio\VideoDepthMaps\<job>\`). ComfyUI is the current executor, not the owner: its node may first save under `D:\MediaStudio\Comfyui\Video\<job>\`, but the accepted deliverables must be moved into the owning project/depth job and must not be retained as a second permanent copy. Never copy the source download into either directory.

## Route and current machine state

Always enter through `$media-studio-orchestrator`, then route here for the depth stage. This Skill does not start ComfyUI for analysis-only work. For generation, reuse `127.0.0.1:8188` or use the user's visible Desktop according to the global ComfyUI rules.

Current status on this workstation is `partial`: the `ComfyUI-Video-Depth-Anything` node is installed, its nodes appear in live `http://127.0.0.1:8188/object_info`, and the Base and Small checkpoints are present. Both models rendered a complete authorized 15.567-second / 467-frame source successfully at 540×960 as grayscale and inferno preview. Do not claim long-video Base quality or production readiness until the segmented route is accepted.

## Research-backed options

The recommended model family for this task is [Video-Depth-Anything](https://github.com/DepthAnything/Video-Depth-Anything), an Apache-2.0 codebase designed for consistent depth on very long videos. Its official README lists relative-depth checkpoints of approximately 28.4M, 113.1M, and 381.8M parameters (Small/Base/Large), with FP16 GPU measurements on A100 of about 6.8 GB, 14 GB, and 23.6 GB respectively. Current model files are approximately 116 MB for Small and 458 MB for Base; verify the Large file size from its current download response before installation. The Small checkpoint is Apache-2.0; the Base and Large checkpoints are listed as CC-BY-NC-4.0, so check the license before commercial use.

For ComfyUI, [ComfyUI-Video-Depth-Anything](https://github.com/yuvraj108c/ComfyUI-Video-Depth-Anything) provides an unofficial node and example workflow; it downloads models under `ComfyUI/models/videodepthanything` and supports Small/Base/Large plus metric variants. [ComfyUI-Depth-Anything-Tensorrt](https://github.com/yuvraj108c/ComfyUI-Depth-Anything-Tensorrt) provides TensorRT image depth nodes and a temporal stabilizer, but its README describes a separate engine-export/model setup and does not by itself prove the long-video Video-Depth-Anything route. [ComfyUI-DepthAnythingV2](https://github.com/kijai/ComfyUI-DepthAnythingV2) is a popular single-image/frame node; use it for quick previews, not as the production temporal route.

The practical choices are:

| Choice | Use | Cost/limitation |
|---|---|---|
| Official Video-Depth-Anything CLI | Best first local route for stable depth video | Separate Python environment and checkpoint; no ComfyUI graph by itself; official requirements pin an older Torch/CUDA-era stack, so isolate it |
| ComfyUI-Video-Depth-Anything | Best if the result must feed a visible ComfyUI workflow | Custom node plus dependencies/models; current repository was tested by its author on a newer RTX/CUDA stack, so run a small smoke before production |
| Depth Anything V2/TensorRT | Fast still/frame preview or existing ComfyUI image workflow | Temporal stability is weaker unless an additional stabilizer is used; not equivalent to VDA |
| Web depth-map service | Quick experiment without local installation | Upload/privacy, quota, unknown model/parameters, and no local reproducibility |

For this RTX 4090, start with Video-Depth-Anything Small or Base in an isolated D-drive environment. Do not install into the video-learning venv or ComfyUI's shared Python until compatibility is checked. Small is the conservative first smoke; Base is the quality comparison; Large is not the first choice because its published FP16 memory figure is close to a 24 GB card's practical ceiling once video buffers and other applications are included.

## Capacity budget before installation

The checkpoint is only part of the installation. Use this planning budget; measure the actual directory after installation because Python wheels and CUDA variants differ.

| Component | Small first smoke | Base comparison | Notes |
|---|---:|---:|---|
| Video-Depth-Anything checkpoint | ~116 MB | ~458 MB | Current official Hugging Face files; Large size must be checked before download |
| Isolated Python environment and CUDA PyTorch wheels | ~3–6 GB | ~3–6 GB | Includes Python packages, PyTorch, torchvision, xformers, OpenCV, decord/imageio, and caches; do not share the video-learning venv |
| Source repository and package caches | ~0.2–1 GB | ~0.2–1 GB | Git checkout is small; pip/Hugging Face caches can be larger |
| One depth-video output | source-dependent | source-dependent | A grayscale H.264 preview is usually similar to or smaller than the source; raw PNG/EXR/NPZ frames can be many times larger |
| Safe working allowance | 8 GB | 10 GB | Includes temporary frames, failed downloads, and one test output; reserve more for long/high-resolution videos |

The official CLI requirements pin `torch==2.1.1`, `torchvision==0.16.1`, `xformers==0.0.23`, and older supporting packages. On this workstation, do not force those pins into an existing environment: Python/CUDA/Torch compatibility must be checked in a new D-drive venv first. The ComfyUI wrapper has a shorter requirements file, but its node code still relies on the ComfyUI Torch runtime and downloads its model under `ComfyUI/models/videodepthanything`; its total incremental footprint is therefore typically the node source plus Python dependencies and one or more checkpoints, not just the checkpoint size.

The ComfyUI node and Base/Small checkpoints are installed. On the RTX 4090, both ran a complete 15.567-second 540×960 source at the same `input_size=518` / `max_res=960` / FP16 configuration: Small about 66.9 seconds, Base about 55.8 seconds. Base remains intended for shorter or segmented jobs. The current local state remains `partial` until visual output review and the full-video segmentation route are established. Inferno preview uses OpenCV `COLORMAP_INFERNO`, avoiding the removed Matplotlib `get_cmap()` API.

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
4. Write the grayscale depth video by default and record model, version/hash, input/output resolution, normalization, near/far polarity, FPS, codec, and elapsed time. An `inferno` color preview is optional and must be explicitly enabled; it is a post-processing view of the same depth result and must not trigger a second model inference.
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
