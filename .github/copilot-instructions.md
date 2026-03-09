# Copilot AI Guard — Review Rules

These rules apply to every pull request targeting `main`. Flag violations clearly and block merge until resolved.

---

## 1. Model Weights & Checkpoints

**Do not commit large model files directly to the repository.**

- Flag any PR that adds or modifies `.safetensors`, `.bin`, `.ckpt`, `.pt`, or `.pth` files.
- These files must live under the `models/` directory structure and be tracked with **Git LFS**, never committed as regular Git objects.
- If a flagged file is under 10 MB it may be intentional (e.g. a small adapter); ask the author to confirm.
- Remind the developer:
  > "Model weights belong in `models/` and must be tracked with Git LFS. Run `git lfs track '*.safetensors'` (or the relevant extension) and commit the updated `.gitattributes` before pushing."

## 2. Gradio Interface Logic (Wan2GP GUI)

**Protect the custom Wan2GP Gradio interface from breaking changes.**

- Scrutinise any diff that touches `app.py` or files containing Gradio block definitions (`gr.Blocks`, `gr.Row`, `gr.Column`, `gr.Group`, `gr.Tab`, `gr.Accordion`).
- Watch for:
  - Removed or renamed Gradio components that other parts of the codebase reference by `elem_id` or variable name.
  - Changes to event handler wiring (`.click`, `.change`, `.submit`, `.select`) that could disconnect existing UI flows.
  - Modifications to `gr.Group` visibility logic — visibility must be stored and toggled on the Group object directly, not on its children (see workspace skill `gradio-group-visibility`).
  - New top-level `gr.Blocks` or `demo.launch()` calls that could conflict with the existing app entry point.
- If a change looks intentional but risky, request the author to confirm the Wan2GP GUI still loads and all tabs render correctly before approving.
