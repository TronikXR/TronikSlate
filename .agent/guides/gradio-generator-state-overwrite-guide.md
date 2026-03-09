---
description: Gradio generator state overwrite — user approve/reject reverts after generation completes
---

## Problem

In Step 3 of Smooth Brain: user approves a shot mid-render (badge turns ✅), but once all renders finish, all shots revert to 🖼 ready. Same with the earlier bug where clicking 👍 sent everything back to pending.

## Cause

The image render generator (`_queue_image_renders`) maintains its own **local copy** of `sb_state`. When the generator yields, it pushes this local copy back into the `gr.State` Gradio component. The generator doesn't see approvals the user made via concurrent button clicks — those write to Gradio state independently. So the generator's final yield **overwrites** those approvals.

## Fix Pattern

```python
def _yield_state(..., preserve_state=False):
    state_out = gr.update() if preserve_state else sb_state  # gr.update() = no-op
    return [status_html, state_out, ...]

# Final yield only — preserve user approvals
yield _yield_state(status, sb_state, stop_btn_visible=False, preserve_state=True)
```

`gr.update()` with no args is a Gradio no-op — the component keeps its current value.

## Key Rules

| Yield type | `preserve_state` | Why |
|---|---|---|
| Mid-render (RENDERING/READY status) | `False` | Need live badge updates |
| Final yield (generation complete) | `True` | Preserve user approvals |
| Early-exit (no shots/no model) | `True` | Nothing changed, don't clobber |

> [!IMPORTANT]
> Any Gradio generator that: (1) adds `gr.State` to its outputs AND (2) the user can click other buttons during generation — MUST use this pattern on its final yield.

## Related

- Skill: `.agent/skills/gradio-generator-state-overwrite/SKILL.md`
- File: `smooth_brain/plugin.py` — `_queue_image_renders()`, `_yield_state()`
