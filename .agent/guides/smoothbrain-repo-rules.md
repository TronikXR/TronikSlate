# Smooth Brain Plugin — Repo Rules

**Source of truth (newest code always here):**
```
f:\pinokio\api\wan.git\app\plugins\smooth_brain\
```
This folder has its own `.git` → `origin = hoodtronik/smoothbrain.git`

**Standalone copy (may lag behind):**
```
f:\pinokio\api\TronikSlate\smooth-brain-wan2gp\
```

## Critical Rules

1. **NEVER copy files from `smooth-brain-wan2gp/` into `smooth_brain/`** — this overwrites newer code
2. Sync direction is always: **plugin dir → standalone copy** (not the other way)
3. Push to GitHub from the plugin's own git:
   ```powershell
   git -C "f:\pinokio\api\wan.git\app\plugins\smooth_brain" push                   # hoodtronik/smoothbrain
   git -C "f:\pinokio\api\wan.git\app\plugins\smooth_brain" push tronikxr main     # TronikXR/smoothbrain
   ```
4. Never copy files INTO the plugin dir to "fix" it — use `git checkout HEAD -- .` instead
5. If `.idx` folder appears in plugin dir, delete it

## What Broke Before
Copying `smooth-brain-wan2gp/plugin.py` (old) → `smooth_brain/plugin.py` (new) wiped out
`describe_character_image`, project persistence, headless render globals → broke Step 2.

See full artifact: `smoothbrain_repo_map.md` in brain folder.
