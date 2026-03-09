# 🔍 TronikSlate Code Audit Report

Audited: March 9, 2026 — Full codebase scan of Python plugin modules + TypeScript web app.

---

## 🔴 Security Issues

### 1. Path Traversal in Server Routes (Medium Risk)

Six server routes construct file paths using user-controlled `req.params` without sanitizing `..` sequences:

| Route | Line | Code |
|---|---|---|
| [rawvideos.ts](file:///f:/pinokio/api/TronikSlate/app/server/routes/rawvideos.ts#L87) | 87 | `path.join(getProjectRawDir(projectId), req.params.filename)` |
| [project.ts](file:///f:/pinokio/api/TronikSlate/app/server/routes/project.ts#L95) | 95 | `path.join(ARCHIVES_DIR, req.params.filename)` |
| [project.ts](file:///f:/pinokio/api/TronikSlate/app/server/routes/project.ts#L274) | 274 | `path.join(PROJECTS_DIR, req.params.id + '.json')` |
| [images.ts](file:///f:/pinokio/api/TronikSlate/app/server/routes/images.ts#L392) | 392 | `path.join(projectImagesDir(projectId), req.params.filename)` |
| [images.ts](file:///f:/pinokio/api/TronikSlate/app/server/routes/images.ts#L408) | 408 | `path.join(projectImagesDir(projectId), req.params.filename)` |
| [audio.ts](file:///f:/pinokio/api/TronikSlate/app/server/routes/audio.ts#L63) | 63 | `path.join(AUDIO_DIR, req.params.filename)` |

> [!WARNING]
> A request like `GET /api/images/project123/../../etc/passwd` could read arbitrary files. Since this is a local-only dev server the real-world risk is low, but it should be fixed.

**Fix**: Add a `sanitizeFilename()` helper that strips path separators and `..` before joining:

```typescript
function sanitizeFilename(name: string): string {
  return path.basename(name); // strips all directory components
}
```

### 2. Unrestricted CORS

[index.ts:34](file:///f:/pinokio/api/TronikSlate/app/server/index.ts#L34) uses `app.use(cors())` with no origin restrictions. Any website can make requests to the API.

**Fix**: Restrict to localhost origins:

```typescript
app.use(cors({ origin: ['http://localhost:5173', 'http://localhost:3001'] }));
```

### 3. Ollama Auto-Installer (Low Risk)

[ollama.py:114](file:///f:/pinokio/api/TronikSlate/ollama.py#L114) runs `OllamaSetup.exe /VERYSILENT` — silently installs software. The download URL is hardcoded from `ollama.com` which is trustworthy, but the installer runs with the current user's privileges.

**Status**: Acceptable since this is documented and expected behavior. The README already warns users.

---

## 🟡 Code Quality Issues

### 4. Duplicated Constants

The same constants are defined in multiple places:

| Constant | Locations |
|---|---|
| `PLUGIN_ID`, `PLUGIN_NAME`, `MAX_CHARS`, etc. | [plugin.py:44-67](file:///f:/pinokio/api/TronikSlate/plugin.py#L44-L67), [constants.py:4-30](file:///f:/pinokio/api/TronikSlate/constants.py#L4-L30) |
| `RESOLUTION_TIERS` | [gpu_utils.py:51](file:///f:/pinokio/api/TronikSlate/gpu_utils.py#L51), [constants.py:23](file:///f:/pinokio/api/TronikSlate/constants.py#L23) |
| `RESO_MAP` / `VIDEO_RESOLUTION` | [plugin.py:54-60](file:///f:/pinokio/api/TronikSlate/plugin.py#L54-L60), [constants.py:14-20](file:///f:/pinokio/api/TronikSlate/constants.py#L14-L20) |

**Fix**: `plugin.py` should import from `constants.py` instead of re-declaring. `constants.py` exists specifically for this purpose but isn't used by the main plugin.

### 5. Hardcoded Server Port

[index.ts:55](file:///f:/pinokio/api/TronikSlate/app/server/index.ts#L55) hardcodes `PORT = 3001`. If another service is using that port, the server fails silently.

**Fix**: Use `process.env.PORT || 3001` or Pinokio's `{{port}}` template.

### 6. Plugin.py Monolith (2182 lines)

While the mixin architecture (`ui_builder.py`, `wiring.py`, `render_engine.py`) exists, [plugin.py](file:///f:/pinokio/api/TronikSlate/plugin.py) at the root still contains ALL the logic inline (2182 lines) rather than importing from the mixins. The mixins appear to exist as a refactored copy that isn't actually used.

**Fix**: Either commit to the mixin architecture (have `plugin.py` import from the mixin files) or consolidate. Currently maintaining two copies means code drift.

### 7. smooth-brain-wan2gp Subfolder Drift

The `smooth-brain-wan2gp/` directory is a standalone plugin distribution with its own copies of `plugin.py`, `ollama.py`, `state.py`, `model_scanner.py`, etc. These have drifted from the root versions (as identified in the earlier analysis).

**Fix**: Use a build/sync script or symlinks to keep these in sync, or generate the distribution from the root files.

---

## 🟢 What's Working Well

### Architecture
- **Zustand stores** (`projectStore`, `modelStore`, `uiStore`, `loraStore`) — clean state management with history support via `zundo`
- **Error boundaries** — every major component wrapped in `<ErrorBoundary>` for graceful degradation
- **SSE streaming** — both the headless render and output watcher use SSE correctly for real-time updates
- **TypeScript types** — well-defined interfaces for `Project`, `Shot`, `ModelDefinition`, etc.

### Python Plugin
- **Offline fallback** — graceful degradation everywhere (Ollama offline → templates, no model → skip refinement)
- **GPU-aware** — resolution tiers and duration limits auto-adjust to VRAM
- **Generator-based rendering** — yields progress per-shot so UI stays responsive
- **Project persistence** — auto-save to JSON with resume support

### Rendering Pipeline
- **Queue.zip format** — portable, self-contained render jobs with embedded images
- **Topological sort** — dependency-aware shot ordering in export
- **Model grouping** — batches shots by model to minimize swaps
- **Resume support** — crashed/cancelled renders can pick up where they left off

---

## Summary

| Severity | Count | Top Items |
|---|---|---|
| 🔴 Security | 2 | Path traversal (6 routes), unrestricted CORS |
| 🟡 Quality | 4 | Duplicated constants, hardcoded port, monolith plugin, subfolder drift |
| 🟢 Strengths | 4 | Clean architecture, robust error handling, GPU-aware defaults, offline fallback |

> [!TIP]
> The highest-impact fix is adding `path.basename()` sanitization to the 6 file-serving routes. The rest are quality improvements that can be addressed incrementally.
