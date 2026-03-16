# Quality Presets Guide

## Background
This pattern was inspired by the LTX-Desktop-WanGP app, which offers "Fast" and "Pro" model tiers. We adapted this into a three-tier system more suited to TronikSlate's workflow.

## How Presets Map to Wan2GP Parameters

### Fast (Lightning LoRA)
Uses `_get_fastest_profile()` to auto-select the best accelerator profile for the current model. The profile typically includes:
- A Lightning LoRA checkpoint (e.g., `Qwen-Image-Edit-Lightning-4steps-V1.0-bf16.safetensors`)
- Reduced step count (usually 4)
- Optimized guidance/flow shift values

### Balanced (25 steps, no LoRA)
Overrides only `num_inference_steps: 25`. All other params (guidance_scale, flow_shift, etc.) come from the model's default settings. No accelerator LoRA is applied.

### High (40 steps, no LoRA) 
Same as Balanced but with `num_inference_steps: 40` for maximum detail.

## Dual Rendering Paths
TronikSlate has two rendering pipelines:

1. **Gradio path** (plugin.py → render_engine.py): Reads `sb_state["quality_preset"]` and applies profile/steps in `_export_videos()`.
2. **React path** (VideoExport.tsx → render.ts → wgp.py subprocess): The frontend builds params directly, setting `num_inference_steps` and clearing LoRA fields client-side before sending to `/api/render/start`.

Both paths must be kept in sync when changing quality preset behavior.

## Adding a New Preset
1. Add entry to `QUALITY_PRESETS` in `model_scanner.py`
2. Update `_export_videos()` in `render_engine.py` if the new preset needs special handling
3. Add the option to the `qualityPreset` union type and button array in `VideoExport.tsx`
4. Update the `handleGenerateVideos` param spread logic in `VideoExport.tsx`
