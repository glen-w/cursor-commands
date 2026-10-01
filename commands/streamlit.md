# Restart Streamlit app (/streamlit)

Kill any `__PROJECT__` Streamlit (or documented UI) process on **port `__UI_PORT__`** and start the project UI.

Skip this command when `__UI_KIND__` is `none`. If `__UI_KIND__` is `flask` (or similar), adapt the start command to `__UI_ENTRY__` but keep the same port and sibling-port rules.

Execute from the workspace root.

**Local sessions only.** This command assumes the user's own machine: it kills and starts a process on a loopback port. In a cloud or remote sandbox, report `skipped (not a local session)` and stop.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| UI kind | `__UI_KIND__` |
| Port | `__UI_PORT__` |
| Entry | `__UI_ENTRY__` |
| Sibling ports never to touch | `__SIBLING_PORTS__` |

**Port policy:** this project always uses **`__UI_PORT__`**. Never inspect, kill, or bind `__SIBLING_PORTS__` — those belong to other local projects and must remain undisturbed.

---

## 1. Kill existing project UI only

- Find the process using port **`__UI_PORT__`** (e.g. `lsof -i :__UI_PORT__` or `lsof -ti :__UI_PORT__`).
- Kill that process only (e.g. `kill $(lsof -ti :__UI_PORT__)`).
- Confirm the port is free.
- **Do not** inspect, kill, or bind `__SIBLING_PORTS__`.

---

## 2. Start the UI

```bash
__UI_ENTRY__
```

Run in the background. Use the documented app entrypoint. Do not start business logic from ad-hoc UI snippets.

---

## 3. Verify

- Optionally check: `curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:__UI_PORT__/` (expect 200).
- Report that the app is running at **http://127.0.0.1:__UI_PORT__/**.
- Note any separate model/runtime dependency (Ollama, Whisper, mail store); UI up does not imply those are ready.

---

## Execution rules

- Run from the workspace root.
- Always use port **`__UI_PORT__`**.
- Never kill or start anything on `__SIBLING_PORTS__`.
- Do not start a second instance on `__UI_PORT__` if one is already running; kill the existing process on that port first.
