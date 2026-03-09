---
name: gradio-generator-state-overwrite
description: Prevent Gradio generator yields from overwriting user-driven state changes (approve/reject during render)
---

# Gradio Generator State Overwrite

## The Problem

When a Gradio generator function (`fn` wired to a `.click()` with `outputs=[..., gr.State, ...]`) yields the full state dict on every iteration, **the last yield wins**. If the user clicks a button (e.g., approve a shot) mid-generation, that button's handler writes to `gr.State`. But when the generator finishes and emits its final yield containing its own local copy of state, it **overwrites** the user's action — reverting approved shots back to ready/pending.

### Symptom
- User approves shot 1 mid-render → badge shows ✅ approved
- Generator finishes all shots → all badges revert to 🖼 ready
- Clicking approve again shows all others as pending (reads stale state)

## Root Cause

Generator holds a **local copy** of state that it mutates. It doesn't see changes made by concurrent button handlers. When it yields the local copy, Gradio replaces the state component's value with it.

## The Fix

Add a `preserve_state` flag to the yield helper. On **intermediate** yields, push the local state (so live status changes like RENDERING/READY appear in the UI). On the **final** yield, emit `gr.update()` (a no-op) for the state slot to preserve any user-driven changes.

```python
def _yield_state(status_html, sb_state, ..., preserve_state=False):
    # ...build badge/button/image updates...
    state_out = gr.update() if preserve_state else sb_state
    return [status_html, state_out, progress, next_btn, ...]

# Intermediate yield — push local state so live status shows
yield _yield_state("🎨 Rendering shot 1...", sb_state, changed_shot=0)

# Final yield — preserve any user approvals made during generation
yield _yield_state("✅ Done!", sb_state, stop_btn_visible=False, preserve_state=True)
```

## Rules

1. **Never overwrite state on final yield** if the user can interact with the UI during generation.
2. **Always add `gr.State` to generator outputs** so intermediate yields can update live status badges.
3. **Use `gr.update()` as a no-op** — it tells Gradio "leave this component as-is".
4. Preserve state only on the **last** yield — intermediate yields must push state so badges stay accurate.

## Applied In

- `smooth_brain/plugin.py` → `_queue_image_renders()` → `_yield_state(preserve_state=True)` on final yield
- Related guide: `.agent/guides/gradio-generator-state-overwrite-guide.md`
