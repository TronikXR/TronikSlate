---
name: model-family-classification
description: How TronikSlate classifies Wan2GP model architectures into UI family groups, and how to avoid substring collision bugs
---

# Model Family Classification

## Problem
TronikSlate scans Wan2GP's `defaults/` and `finetunes/` folders for model JSON files and groups them into families for the UI dropdown. The `inferFamily()` function in `app/server/routes/models.ts` determines the family using architecture string matching.

## Critical Pattern: Substring Collisions

Architecture strings can contain substrings that match **other** families. Example:
- `ltx2_22B` contains `2_2` → falsely matched Wan 2.2 before LTX check runs

### Rule: Order checks from most-specific to least-specific
```
1. Explicit `model.group` from JSON (always wins)
2. Specific prefix checks (ltx, hunyuan_1_5, hunyuan, k5_, longcat, ovi)
3. Wan 2.2 suffix check (regex: /_2_2(_|$)/ — NOT includes('2_2'))
4. Non-video arch filter
5. WAN_21_ARCHITECTURES explicit allowlist
6. Smart fallback for unknown architectures (extract prefix)
```

### Key Files
- **Family inference**: `app/server/routes/models.ts` → `inferFamily()`
- **Non-video filter**: `NON_VIDEO_ARCH_PREFIXES` array
- **Wan 2.1 allowlist**: `WAN_21_ARCHITECTURES` Set
- **Family labels**: `getFamilyLabel()` — auto-generates labels for unknown families

## When Adding New Models
1. If Wan2GP adds a new model with an explicit `group` field in its JSON, no code changes needed
2. If the model has a **novel architecture prefix** (e.g. `mochi_v2`), the smart fallback will create a new family called "Mochi" automatically
3. If the model should be in Wan 2.1 but has a new architecture name, add it to `WAN_21_ARCHITECTURES`
4. If the model is non-video (TTS, image-only), add its prefix to `NON_VIDEO_ARCH_PREFIXES`

## Verification
After any changes, test the `/api/models` endpoint and check `families` array for correct counts. Look for unexpected new families — they indicate missing entries in the allowlist.
