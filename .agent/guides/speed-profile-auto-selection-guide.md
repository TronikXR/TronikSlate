---
description: Speed profile auto-selection — replicate TronikSlate's accelerator profile ranking in plugin code
---

## Problem

Video renders in Smooth Brain plugin used raw model defaults instead of applying accelerator profiles (speed loras, reduced steps) that TronikSlate auto-selects.

## How TronikSlate Does It

1. Fetches profiles from `/api/models/profiles/{model}` (or `scan_profiles()` locally)
2. Sorts by: **i2v preference** → **latest date** → **lowest step count**
3. Auto-applies the top-ranked profile's params to every video render task

## Plugin Implementation

```python
def _get_fastest_profile(self, model_id):
    profiles = scan_profiles(model_id)
    # Sort: i2v match → latest date → lowest steps
    ...
    return ranked[0]  # {name, params}

# In _export_videos:
profile = self._get_fastest_profile(video_model)
if profile:
    params.update(profile["params"])
```

## Key Details

- `scan_profiles()` reads `profiles/<arch>/*.json` files from wan2gp app dir
- Profile params include `num_inference_steps`, `lset_name`, `activated_loras`, `guidance_scale`, etc.
- Profile params are applied AFTER `build_video_params()` so they override model defaults
- `image_start` must be set AFTER profile params to avoid being overwritten

## Related

- `model_scanner.py` → `scan_profiles()`
- `plugin.py` → `_get_fastest_profile()`, `_export_videos()`
