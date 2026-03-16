# Quality Preset Feature — Walkthrough

## What Changed

Added a **Fast / Balanced / High** quality preset selector to TronikSlate's video generation pipeline, inspired by the LTX-Desktop-WanGP app.

| Preset | Steps | LoRA | Behavior |
|---------|-------|------|----------|
| ⚡ **Fast** | Profile-defined (~4) | Lightning LoRA | Uses existing accelerator profile (current default) |
| ⚖️ **Balanced** | 25 | None | Clean render, no accelerator |
| 💎 **High** | 40 | None | Maximum quality, slowest |

## Files Modified

### Python Backend

#### [model_scanner.py](file:///F:/pinokio/api/TronikSlate/model_scanner.py)
render_diffs(file:///F:/pinokio/api/TronikSlate/model_scanner.py)

Added `QUALITY_PRESETS` dict defining the three tiers.

---

#### [render_engine.py](file:///F:/pinokio/api/TronikSlate/render_engine.py)
render_diffs(file:///F:/pinokio/api/TronikSlate/render_engine.py)

Replaced unconditional `_get_fastest_profile()` call with preset-aware logic. Reads `sb_state["quality_preset"]` (defaults to "fast"). For Balanced/High, sets fixed step count and skips LoRA.

---

#### [state.py](file:///F:/pinokio/api/TronikSlate/state.py)
render_diffs(file:///F:/pinokio/api/TronikSlate/state.py)

Added `quality_preset: str = "fast"` to `SmoothBrainSession` for project persistence.

---

### React Frontend

#### [VideoExport.tsx](file:///F:/pinokio/api/TronikSlate/app/src/components/SmoothBrain/VideoExport.tsx)
render_diffs(file:///F:/pinokio/api/TronikSlate/app/src/components/SmoothBrain/VideoExport.tsx)

- New `qualityPreset` state variable
- 3-button toggle UI (cyan/purple/amber color-coded) between Auto/Manual toggle and Duration slider
- When Balanced or High is selected, `num_inference_steps` is overridden (25 or 40) and LoRA settings are cleared in the render params
- Preset name shown in the info bar

## Verification

| Check | Result |
|-------|--------|
| `tsc --noEmit` | ✅ Clean |
| Python `py_compile` (3 files) | ✅ Clean |
| Manual UI test | ⏳ Pending user |
