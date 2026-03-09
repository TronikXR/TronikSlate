---
name: gradio-resume-state-restoration
description: Ensure Gradio resume/load functions restore ALL UI components — not just state dicts
---

# Gradio Resume State Restoration

## The Problem

When a Gradio app has a "resume" function that loads saved state from disk (e.g., project JSON), the function's `outputs=[]` list must include **every** UI component that needs updating. Missing components stay blank even though the data exists in the loaded state dict.

### Symptom
- Resume loads correctly → state dict has all data
- But some panels/images/galleries are empty until user navigates away and back
- Navigating forward then back populates them because the step transition functions have the right output lists

## Root Cause

Each Gradio step transition function (e.g., `_enter_step4`) has its own `outputs=[]` list wired to its button click. These lists include all the components that step needs. But the resume function typically only includes Step 1 outputs, not Step 3/4 panels.

## The Fix

The resume function must include outputs for ALL steps and conditionally populate them:

```python
# Wiring
self.sb_resume_btn.click(
    fn=self._resume_project,
    outputs=[
        self.sb_state,
        # Step 1 outputs...
        *step_outputs,
        # Step 2 outputs
        self.sb_char_image,
        # Step 3 outputs
        self.sb_progress_html,
        self.sb_step3_next,
        *self._storyboard_panel_outputs(),
        # Step 4 outputs
        *self._all_vid_panel_outputs(),
        self.sb_video_gallery,
    ],
)

# Inside _resume_project:
if step >= 4:
    vid_updates = self._make_video_panel_updates(data)
    vid_gallery = self._refresh_gallery(data, "videos")
else:
    vid_updates = [gr.update()] * (3 * len(self.sb_video_panels))
    vid_gallery = gr.update()
```

Use `gr.update()` (no-op) for components that don't need updating at the current step.

## Rules

1. **Every component you want populated on resume must be in `outputs=[]`**
2. **Use `gr.update()` for components that don't apply at the current step** (it's a no-op)
3. **Error fallback count must match** — if the function can return early with `[gr.update()] * N`, compute N dynamically
4. **Reuse existing helpers** like `_make_storyboard_updates()` and `_make_video_panel_updates()` rather than rebuilding update logic

## Applied In

- `smooth_brain/plugin.py` → `_resume_project()` — restores Steps 2, 3, and 4 panels on resume
