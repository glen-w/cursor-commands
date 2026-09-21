# Docker rebuild (# rebuild)

Rebuild the image and launch using docker compose — **only when this repo has Docker packaging**.
Execute from the workspace root.

If there is no `Dockerfile` / `docker-compose.yml` (greenfield / local-venv-only), report that and stop after optional `# backup`; do not invent Docker assets unless the user asked.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Compose service | `__COMPOSE_SERVICE__` |
| UI URL | `http://127.0.0.1:__UI_PORT__/` |
| Sibling ports never to touch | `__SIBLING_PORTS__` |
| Notes | Do not bake model weights into the image |

---

## 0. Run backup first (mandatory)

Run `# backup`. Wait for it to complete, then proceed.

---

## 1. Tear down

Tear-down is **disabled** for safety after repeated data loss.

- **Do not run `docker compose down`** unless the user explicitly requests it. Report that tear-down is disabled.

---

## 2. Prune

Prune is **disabled** for safety after repeated data loss.

- **Do not run** `docker builder prune`, `docker image prune`, or `docker system prune`. Report that prune steps are disabled.

---

## 3. Build

- Rebuild the image (no cache): `docker compose build --no-cache` (or `docker build` if only a Dockerfile exists).
- Do **not** bake model weights into the image; models stay on the host or a mounted volume.

---

## 4. Launch

- Prefer documented compose service names from the card.
- Open the UI at **http://127.0.0.1:__UI_PORT__/**.
- **Never inspect, bind, or kill `__SIBLING_PORTS__`.**
- Confirm any required sidecar (e.g. Ollama) separately; a running UI does not imply models are loaded.

---

## Execution rules

- Run all steps from the workspace root where `docker-compose.yml` / `Dockerfile` lives.
- After completion, confirm that the container starts and the entrypoint works.
- Never delete mounted project directories or host data as part of rebuild.
