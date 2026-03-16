---
name: quality-presets
description: How TronikSlate implements Fast/Balanced/High quality presets for video generation — mapping user-facing labels to step counts and LoRA strategies
---

# Quality Presets for Video Generation

## Overview
TronikSlate offers users three quality tiers for video generation instead of exposing raw inference step counts:

| Preset | Steps | LoRA | Constant Key |
|---------|-------|------|-------------|
| **Fast** | Profile-defined (~4) | Lightning accelerator LoRA | `use_profile: True` |
| **Balanced** | 25 | None | `num_inference_steps: 25` |
| **High** | 40 | None | `num_inference_steps: 40` |

## Where It Lives

### Constants
`model_scanner.py` → `QUALITY_PRESETS` dict:
```python
QUALITY_PRESETS = {
    "fast":     {"use_profile": True,  "num_inference_steps": None},
    "balanced": {"use_profile": False, "num_inference_steps": 25},
    "high":     {"use_profile": False, "num_inference_steps": 40},
}
```

### Backend Logic
`render_engine.py` → `_export_videos()` reads `sb_state["quality_preset"]` (default: `"fast"`):
- **Fast**: calls `_get_fastest_profile()` → applies Lightning LoRA profile params
- **Balanced/High**: sets `num_inference_steps` directly, no LoRA

### Frontend
`VideoExport.tsx` → `qualityPreset` state + 3-button toggle. When Balanced/High is selected, the render params explicitly set `num_inference_steps` and clear LoRA fields (`activated_loras: []`, `loras_multipliers: ''`, `lset_name: ''`).

## The Rule
When adding new quality presets or modifying the mapping:
1. Update `QUALITY_PRESETS` in `model_scanner.py`
2. Ensure both rendering paths handle the new preset (Gradio `render_engine.py` AND React `VideoExport.tsx`)
3. The preset value is persisted in `state.py` `SmoothBrainSession.quality_preset`

## Anti-pattern
```python
# ❌ BAD — hardcoding steps without checking quality_preset
profile_params = self._get_fastest_profile(video_model)

# ✅ GOOD — respect quality preset selection
preset_cfg = QUALITY_PRESETS.get(quality_preset, QUALITY_PRESETS["fast"])
if preset_cfg["use_profile"]:
    profile = self._get_fastest_profile(video_model)
    ...
else:
    profile_params = {"num_inference_steps": preset_cfg["num_inference_steps"]}
```
