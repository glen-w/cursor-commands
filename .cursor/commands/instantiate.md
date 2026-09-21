# Instantiate templates into a project (# instantiate)

Copy the slash-command templates from this repo into a consuming project's `.cursor/commands/`, then fill every `__TOKEN__` from that project's card. Do not leave sentinels behind.

This command belongs to this template repo. Do not copy `instantiate.md` into product repos.

---

## Inputs

1. **Target repo** (required): path the user names. If unclear, ask once and stop.
2. **Project card**: `.cursor/project-card.md` in the target if present; otherwise infer from `pyproject.toml`, `AGENTS.md`, existing `.cursor/commands/`, and [project-card.example.md](../../project-card.example.md). Confirm inferred ports and backup excludes with the user when they would be destructive if wrong.

---

## What to do

1. Read [AGENTS.md](../../AGENTS.md) and [commands/_placeholders.md](../../commands/_placeholders.md).
2. List existing `.cursor/commands/*.md` in the target. **Do not overwrite** a filled command unless the user asked to replace it. New files only by default; report skipped names.
3. Copy every `commands/*.md` that does **not** start with `_` into `$TARGET/.cursor/commands/`. Skip this file (`instantiate.md`).
4. Skip `streamlit.md` when `__UI_KIND__` is `none`. Skip `rebuild.md` and `dockerfile-efficiency.md` when there is no Dockerfile / compose file **unless** the user asked to keep the stubs.
5. Fill the Project card tables and replace all `__TOKEN__` values. Grep the target `.cursor/commands/` for `__` and fix any leftovers.
6. Keep standing policies intact (hub backup under `$HOME/Documents/code backups` unless the card sets `sibling`, no `/Users/` literals, cleanup/prune/compose-down disabled, pre-release is local confidence). Fill `__BACKUP_HUB__` from the card (default `$HOME/Documents/code backups`).
7. Ensure the target `.gitignore` lists `.cursor/commands/` (create or append if missing). Do not ignore this template repo's own `.cursor/commands/`.
8. Report: files added, files skipped, tokens filled, any card values that were guessed.

---

## Execution rules

- Do not modify product application code.
- Do not commit unless the user asked.
- Do not invent a new slash command in the target.
