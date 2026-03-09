---
name: base-model-type-params
description: Always set both model_type AND base_model_type when creating shots — missing base_model_type causes wrong model usage
---

# base_model_type Must Always Be Set

## Problem
When creating shots (from Skills page, LoRA panel, or programmatically), both `model_type` and `base_model_type` must be set in the shot's `params`. If `base_model_type` is missing, the `BASE_GENERATION_PARAMS` default (`ltx2_19B`) leaks through during render export, causing wan2gp to use the wrong model.

## Root Cause
`BASE_GENERATION_PARAMS` in `app/src/types/project.ts` hardcodes:
```typescript
model_type: 'ltx2_distilled',
base_model_type: 'ltx2_19B',
```
The render pipeline merges shot params onto these defaults. If a shot only sets `model_type` but not `base_model_type`, the export ZIP's `queue.json` will contain the wrong architecture.

## The Rule
**Every code path that sets `model_type` MUST also set `base_model_type`.**

### Affected locations:
1. **`LoraPanelPreview.handleAddShot()`** in `SkillsPage.tsx` — sets both
2. **`buildShotFromSkill()`** in `skillEngine.ts` — sets `base_model_type: route.baseModelType`
3. **`handleModelChange()`** in `GlobalParamsEditor.tsx` — sets `base_model_type: model.architecture`
4. **`handleShotModelChange()`** in `ParamsEditor.tsx` — sets `base_model_type: model.architecture`

### How to verify
Check `queue.json` inside the exported ZIP — `base_model_type` should match the intended model architecture, not `ltx2_19B` (unless that's actually the intended model).

## Anti-pattern
```typescript
// ❌ BAD — base_model_type will default to 'ltx2_19B'
params: { model_type: lora.defaultModelType }

// ✅ GOOD — explicitly set both
params: {
    model_type: lora.defaultModelType,
    base_model_type: lora.defaultModelType,
}
```
