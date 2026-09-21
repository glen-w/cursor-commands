# Dockerfile efficiency (# dockerfile-efficiency)

Assess and improve Docker image size and build hygiene using a structured diagnosis and checklist. Do not change behavior; focus on size and hygiene.

Execute from the workspace root.

If no Dockerfile exists yet, summarize that Docker packaging is out of scope for now and stop (do not create Dockerfiles unless the user asked).

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Image names / tags | `__IMAGE_NAMES__` |
| Data / model dirs to keep out of layers | `__BACKUP_EXCLUDES__` plus `models/`, caches |
| Notes | Do not bake Ollama/GGUF/vision/Whisper weights into the app image |

---

## 0. Run backup first (mandatory)

Run `# backup`. Wait for it to complete, then proceed.

---

## 1. Quick diagnosis

- `docker system df` — images vs build cache vs volumes.
- `docker images --digests` — spot the real offenders.
- `docker history <image>:<tag>` — find the layer that adds the big chunk.
- Record a baseline (sizes, layer counts) before making changes.

---

## 2. Dockerfile hygiene

- **Base:** small runtime base (e.g. `python:3.11-slim` or `debian:bookworm-slim`).
- **Stages:** keep build tools only in the builder stage. In runtime, install only what you need (no compilers).
- **apt:** `--no-install-recommends`. Clean apt cache in the same layer: `rm -rf /var/lib/apt/lists/*`.
- **pip:** `pip install --no-cache-dir ...`. Do not copy a local venv into the image.
- **Current setup:** note which Dockerfiles exist. Document the role of each; remove or update references to removed variants.
- **Models / sidecars:** do not install model runtimes or pull weights inside the app image by default; talk to a host/service endpoint or a mounted volume.

---

## 3. .dockerignore

Exclude: `.git/`, `.venv/`, `venv/`, `__pycache__/`, `.pytest_cache/`, `.mypy_cache/`, `dist/`, `build/`, `.ruff_cache/`.

Exclude large local data folders from the card (outputs, runs, models, tmp, project data).

---

## 4. Model / data strategy (biggest wins)

- Do not bake model weights into the image unless you truly must.
- Mount caches at runtime (`HF_HOME`, `TORCH_HOME`, project `models/`) rather than copying them into layers.
- Keep test fixtures and user corpora out of the production image.

---

## 5. Layer / cache hygiene

- **Do not run prune.** Report that `docker builder prune`, `docker image prune`, and `docker system prune` are disabled for safety after repeated data loss.
- Check volumes read-only: `docker volume ls` / `docker volume inspect`.
- Document when one would run prune so disk usage stays predictable (user may run manually if desired).

---

## 6. Implementation order

Diagnosis → Dockerfile hygiene → `.dockerignore` → sidecar images if maintained → docs.

---

## Execution rules

- Do not change runtime behavior.
- After completion, summarize: baseline vs after, what was changed, and any remaining recommendations.
