---
description: Gradio resume must include all step outputs — blank panels on resume despite loaded state
---

## Problem

Resuming a project shows the correct step but some panels are empty (e.g., Step 4 video cards have no prompts, Step 2 character image is blank). Navigating back then forward populates them.

## Cause

The resume function's `outputs=[]` only included Step 1-3 components. Step 4 video panels were not in the output list, so `_make_video_panel_updates()` was never called on resume.

## Fix

Add every step's outputs to the resume button wiring. Conditionally populate based on `current_step`:

```python
# Step 4 — only populate if resuming at step 4+
if step >= 4:
    vid_panel_updates = self._make_video_panel_updates(data)
    vid_gallery = self._refresh_gallery(data, "videos", [".mp4"])
else:
    vid_panel_updates = [gr.update()] * (3 * len(self.sb_video_panels))
    vid_gallery = gr.update()
```

## Checklist for Future Resume Changes

| Step | Components to include | Helper to call |
|---|---|---|
| 1 | concept, shot_count, vibe, roll_status | Direct dict.get() |
| 2 | char_image | Load from character_images array |
| 3 | progress_html, step3_next, storyboard panels | `_make_storyboard_updates()` |
| 4 | video panels, video gallery | `_make_video_panel_updates()` |

> [!IMPORTANT]
> When adding a new step or new components to any step, update BOTH the step transition function AND the resume function outputs.

## Related

- Skill: `.agent/skills/gradio-resume-state-restoration/SKILL.md`
- Skill: `.agent/skills/gradio-generator-state-overwrite/SKILL.md`
