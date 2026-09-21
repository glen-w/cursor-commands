# Backup workspace (# backup)

Back up **code** from the workspace to a date-stamped zip, excluding gitignored and library/cache content. Mirror Cursor commands beside the zip.

Execute from the repository root. Archives go under `$HOME/Documents/code backups/<repo-name> backup/` — not a sibling of the repo. Do **not** hard-code `/Users/…` literals; use `$HOME` / `$BACKUP_HUB`.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Staging prefix | `__STAGING_PREFIX__` |
| Extra excludes | `__BACKUP_EXCLUDES__` |
| Extra includes | `__BACKUP_INCLUDES__` |
| Verify paths (when they exist) | `__VERIFY_PATHS__` |
| Backup hub | `__BACKUP_HUB__` (default `$HOME/Documents/code backups`) |
| Backup root | hub `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"`. Only use sibling `"$REPO_ROOT backup"` when the project card sets `backup_hub: sibling`. |

---

## What to do

1. **Resolve backup path and date**
   - `REPO_ROOT` = current workspace / git root (`$(git rev-parse --show-toplevel 2>/dev/null || pwd)`)
   - Hub: `__BACKUP_HUB__` (default `$HOME/Documents/code backups` — one folder for every project's code backups)
   - Base directory: `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"` (e.g. `.../code backups/myapp backup`)
   - Archive name: `YYMMDD.zip` (e.g. `260902.zip`). Use `date +%y%m%d`
   - Full destination: `"$BACKUP_ROOT/YYMMDD.zip"`
   - **Sibling override:** if the card sets `backup_hub: sibling`, set `BACKUP_ROOT="${REPO_ROOT} backup"` instead and skip `BACKUP_HUB`.

2. **Create destination**
   - Create the base directory if it does not exist.
   - **Do not overwrite** an existing zip; if `YYMMDD.zip` already exists, use `YYMMDD-HHMM.zip`.

3. **Create zip (code only)**
   - Stage filtered files to a temp directory, then zip and remove the staging dir.
   - **Goal:** source, config, docs, tests, and synthetic fixtures — not user data, models, generated artifacts, or live copies.
   - **Expected size:** small (typically a few MB compressed). Large zips mean a data dir leaked in.
   - **Default foundational paths (include when they exist):** `src/`, `tests/`, `scripts/`, `docs/`, `assets/`, root manifests (`pyproject.toml`, `requirements*.txt`, `uv.lock`, `Dockerfile`, `docker-compose*.yml`, `Makefile`, `pytest.ini`, `.env.example`, `NOTICE`, `LICENSE`, `MANIFEST.in`), and `.cursor/commands/`.
   - **Always exclude:**
     - Project data / output dirs from the card (`__BACKUP_EXCLUDES__`)
     - `.git`, `node_modules`, `__pycache__`, `.venv`, `venv`, `env`, `.env`
     - `.pytest_cache`, `.mypy_cache`, `.ruff_cache`
     - `*.pyc`, `*.pyo`, `*.egg-info`, `dist/`, `build/`
     - Model / weight files: `*.pt`, `*.pth`, `*.bin`, `*.onnx`, `*.safetensors`, `*.gguf`
     - `.DS_Store`, `*.log`, `*.tmp`, `.coverage`, `coverage.xml`, `coverage.json`, `htmlcov/`
   - Also apply `.gitignore` via `--exclude-from` when that file exists; keep explicit includes so gitignored-but-wanted paths (e.g. `.cursor/`) still land in the zip.
   - **Recipe** (run from repository root; replace the extra exclude/include/verify lines from the card):

     ```bash
     REPO_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
     BACKUP_HUB="${BACKUP_HUB:-__BACKUP_HUB__}"
     BACKUP_ROOT="$BACKUP_HUB/$(basename "$REPO_ROOT") backup"
     STAMP=$(date +%y%m%d)
     ZIP_PATH="$BACKUP_ROOT/${STAMP}.zip"
     if [ -f "$ZIP_PATH" ]; then
       STAMP=$(date +%y%m%d-%H%M)
       ZIP_PATH="$BACKUP_ROOT/${STAMP}.zip"
     fi
     mkdir -p "$BACKUP_ROOT"
     STAGING=$(mktemp -d "${TMPDIR:-/tmp}/__STAGING_PREFIX__.XXXXXX")
     RSYNC_EXCLUDES=(
       --exclude='.git'
       --exclude='node_modules'
       --exclude='__pycache__'
       --exclude='.venv'
       --exclude='venv'
       --exclude='env'
       --exclude='.pytest_cache'
       --exclude='.mypy_cache'
       --exclude='.ruff_cache'
       --exclude='*.pyc'
       --exclude='*.egg-info'
       --exclude='dist'
       --exclude='build'
       --exclude='*.pt'
       --exclude='*.pth'
       --exclude='*.bin'
       --exclude='*.onnx'
       --exclude='*.safetensors'
       --exclude='*.gguf'
       --exclude='.DS_Store'
       --exclude='.coverage'
       --exclude='coverage.xml'
       --exclude='coverage.json'
       --exclude='htmlcov'
       # extra from project card — one --exclude= per __BACKUP_EXCLUDES__ entry
     )
     RSYNC_INCLUDES=(
       --include='src/***'
       --include='tests/***'
       --include='scripts/***'
       --include='docs/***'
       --include='assets/***'
       --include='.cursor/***'
       --include='*.md'
       --include='*.toml'
       --include='*.txt'
       --include='*.yml'
       --include='*.yaml'
       --include='*.ini'
       --include='*.sh'
       --include='*.example'
       --include='*.lock'
       --include='Makefile'
       --include='Dockerfile'
       --include='LICENSE'
       --include='NOTICE'
       --include='MANIFEST.in'
       --include='.coveragerc'
       --include='.dockerignore'
       --include='.gitignore'
       # extra from project card — one --include= per __BACKUP_INCLUDES__ entry
       --exclude='*'
     )
     if [ -f .gitignore ]; then
       rsync -a "${RSYNC_EXCLUDES[@]}" "${RSYNC_INCLUDES[@]}" --exclude-from='.gitignore' . "$STAGING/"
     else
       rsync -a "${RSYNC_EXCLUDES[@]}" "${RSYNC_INCLUDES[@]}" . "$STAGING/"
     fi
     (cd "$STAGING" && zip -rq "$ZIP_PATH" .)
     rm -rf "$STAGING"
     for req in __VERIFY_PATHS__; do
       if [ -e "$REPO_ROOT/$req" ] || [ -e "$REPO_ROOT/${req%/}" ]; then
         unzip -l "$ZIP_PATH" | grep -q "$req" || { echo "BACKUP VERIFY FAILED: missing $req in $ZIP_PATH"; exit 1; }
       fi
     done
     ```

4. **Back up Cursor custom commands beside the zip**
   - `mkdir -p "$BACKUP_ROOT/custom-commands"`
   - `rsync -a .cursor/commands/ "$BACKUP_ROOT/custom-commands/"` from repository root.
   - Latest command `.md` files live at `"$BACKUP_ROOT/custom-commands/"` for restore or inspection.

5. **Confirm**
   - Report: zip path, file size (`ls -lh`), and success. If staging/zip fails, report the error and remove any leftover staging dir.

---

## Execution rules

- Run from repository root (`REPO_ROOT`).
- Do not delete or modify the existing workspace; only create the zip and update `custom-commands/`.
- Do not include secrets, `.env` values, or live user data.
