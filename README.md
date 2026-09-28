<p align="center">
  <img src="logo.png" alt="Cursor" width="160"/>
</p>

# cursor-commands

Plain-Markdown **project slash commands** for [Cursor](https://cursor.com). Copy the templates you need into a product repo, fill the placeholders, and keep that filled copy local.

Cursor loads one file per command from `<project>/.cursor/commands/`. The filename is the slash name (`backup.md` → `/backup`). There is no YAML frontmatter: Cursor injects the whole file as the prompt, so each command carries its own project card.

This repository is prompts, not an application. Filled copies belong in the product and should stay gitignored there, because they name ports, fixtures, and probe recipes.

License: [MIT](LICENSE).

## Use cases

- Start a repo and want the same backup, test, cleanup, and docs prompts without rewriting them.
- Run a local confidence check before a release. `/pre-release` reports `pass`, `warning`, `failure`, or `skipped`. It does not tag or push.
- Check a diff without the full harden path. `/light-test` runs the tests that cover the changed files and skips large fixtures, live models, and heavy extras.
- Keep Docker and Streamlit restarts from pruning images, running `compose down`, or touching another project's port.
- Copy a handful of representative source files out for review, without tests, fixtures, or docs.

These templates assume a Python tree (`pytest`, and usually `ruff` / `black` / `mypy`) with optional Docker or Streamlit. If the product uses other tools, change the card commands. Do not leave the sample ports or fixture paths in place.

## Use in a project

From this repo:

```bash
mkdir -p /path/to/project/.cursor/commands
find commands -maxdepth 1 -name '*.md' ! -name '_*' \
  -exec cp {} /path/to/project/.cursor/commands/ \;
```

Or open this repo in Cursor and run `/instantiate`, pointing at the product checkout.

Then:

1. Add `.cursor/commands/` to the product `.gitignore` if it is not already ignored.
2. Copy [project-card.example.md](project-card.example.md) into the product as `.cursor/project-card.md` and replace the sample values.
3. Paste those values into each command's **Project card** and replace every `__TOKEN__`. The token list is [commands/_placeholders.md](commands/_placeholders.md). A command with a leftover `__…__` is not ready.

Skip files the product does not need. See below.

## What to copy

| File | Slash | When | Mutating? |
| --- | --- | --- | --- |
| [backup.md](commands/backup.md) | `/backup` | Code-only zip under the backup hub, plus a `custom-commands/` mirror | zip only |
| [tests.md](commands/tests.md) | `/tests` | Review then expand the offline suite | tests-only (backup first) |
| [deep-test.md](commands/deep-test.md) | `/deep-test` | After a plan: landing → tests → small/large/extra probes → pre-release | yes, targeted fixes |
| [light-test.md](commands/light-test.md) | `/light-test` | Change set only: focused offline tests; no large, LLM, or heavy-dependency runs | yes, targeted fixes |
| [watch.md](commands/watch.md) | `/watch` | Babysit a long run; fix/hang-recover; log under `assessments/` | yes, targeted fixes |
| [pre-release.md](commands/pre-release.md) | `/pre-release` | Local confidence report | no (report only) |
| [refactor.md](commands/refactor.md) | `/refactor` | Assessment; no code changes | no |
| [export-exemplars.md](commands/export-exemplars.md) | `/export-exemplars` | Copy ≤10 representative files to `~/Desktop/<topic>/` | copy only |
| [cleanup.md](commands/cleanup.md) | `/cleanup` | Format / lint / types. Deletes stay disabled | format/lint/types |
| [docs.md](commands/docs.md) | `/docs` | CONTRACT / GUIDE / ARCHITECTURE / PRODUCT pass | docs only |
| [rebuild.md](commands/rebuild.md) | `/rebuild` | `compose build --no-cache` + launch. Down/prune disabled | image rebuild |
| [streamlit.md](commands/streamlit.md) | `/streamlit` | Restart UI on the project port; never touch sibling ports | process restart |
| [dockerfile-efficiency.md](commands/dockerfile-efficiency.md) | `/dockerfile-efficiency` | Image size/hygiene. Prune disabled | Dockerfile only |

Skip `streamlit.md` when there is no UI. Skip `rebuild.md` and `dockerfile-efficiency.md` when there is no Dockerfile or compose file. Shrink or skip `docs.md` when the product is README-only.

`export-exemplars.md` has no tokens. The others do.

## Standing policies

These belong in every filled copy. Do not weaken them for one project.

1. Resolve paths from `REPO_ROOT`. No `/Users/…` literals. Use `$HOME` / `$BACKUP_HUB`.
2. Backup root defaults to `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"` with `$BACKUP_HUB=$HOME/Documents/code backups`. Sibling `"$REPO_ROOT backup"` only when the card sets `backup_hub: sibling`.
3. Code-only zips. Include `.cursor/commands/` in the zip and mirror them to `"$BACKUP_ROOT/custom-commands/"`.
4. Mutating commands run `/backup` first. Nested backups skip when `/deep-test` or `/light-test` already ran it.
5. Artifact cleanup, `docker compose down`, and docker prune stay disabled.
6. `/pre-release` is local confidence: `pass` / `warning` / `failure` / `skipped`. Never a silent pass. Never tag or push.
7. Default tests stay fast and offline. `/light-test` stays on that path for the change set only: no large fixtures, live models, Docker, or heavy optional stacks.
8. Do not publish, push, or tag unless the user explicitly asks.
9. Secrets stay out of git and out of zips.

## Layout

```
commands/                 templates (copy these)
  _placeholders.md        token list; do not copy
.cursor/commands/         /instantiate for this repo only
project-card.example.md   example card
AGENTS.md                 agent instructions for this repo
LICENSE
```

Files whose names start with `_` are not slash commands.

## Related

Cursor still loads `.cursor/commands/*.md`. These files are project commands, not user-global `~/.cursor/commands/`, and not Agent Skills.

<p align="center">
  <a href="https://ko-fi.com/C0C1XK8G" target="_blank" rel="noopener noreferrer"><img height="36" style="border:0;height:36px" src="https://storage.ko-fi.com/cdn/kofi6.png?v=6" alt="Buy Me a Coffee at ko-fi.com" /></a>
</p>
