# AGENTS.md

Instructions for agents working in this template library. It is prompts, not a product. Do not invent a product, stack, or biography here.

## Read order

1. [README.md](README.md)
2. This file
3. [commands/_placeholders.md](commands/_placeholders.md)
4. Then the task

## What this repo is

Project-agnostic slash commands for coding agents. They run unchanged in Cursor and Claude Code. Consuming projects keep **filled** copies under their own commands directory (`.cursor/commands/` or `.claude/commands/`). This repo stays generic. Do not merge product-specific pytest file lists, fixture paths, or ports back into `commands/` unless they belong in the project-card tokens.

## Stance

Push back on:

- A new command that is a one-off (one project, one sitting)
- Re-enabling `/cleanup` deletes, `docker compose down`, or docker prune inside a template
- Hard-coded `/Users/…` paths
- Treating `/pre-release` as the tag gate
- Dual-maintaining the same command as both a skill and a slash command here (slash commands stay `.md` under `commands/`)
- Wording that only one agent platform understands (a named IDE feature, a platform-only tool). Describe the behaviour instead

## Layout

```
commands/                 # templates to copy → <project>/<commands directory>/
  _placeholders.md        # token list; do not copy
  *.md                    # one file per slash command; filename = /name
.cursor/commands/         # commands for *this* repo only
  instantiate.md          # pointer to the Instantiate SOP below
.claude/commands/
  instantiate.md          # same pointer, for Claude Code
project-card.example.md
LICENSE
```

Files whose names start with `_` are not slash commands.

## Platforms

| Platform | `__COMMANDS_DIR__` | Project card | Target `.gitignore` line |
| --- | --- | --- | --- |
| Cursor | `.cursor/commands` | `.cursor/project-card.md` | `.cursor/commands/` |
| Claude Code | `.claude/commands` | `.claude/project-card.md` | `.claude/commands/` |

The command file is the same on both. Only the directory differs.

## Instantiate (SOP)

Trigger: “instantiate”, “copy commands into X”, `/instantiate`. This is the only copy of the procedure; both `instantiate.md` files point here.

**Inputs**

1. **Target repo** (required): a git project the user names. If unclear, ask once and stop.
2. **Platform**: the user's choice when they state one. Otherwise detect it from the target: `.cursor/` present means Cursor, `.claude/` present means Claude Code. If both or neither exist, ask once. One platform per run. Take the commands directory, card path, and `.gitignore` line from the Platforms table.
3. **Project card**: the card file in the target if present; otherwise infer from `pyproject.toml`, `AGENTS.md`, existing commands, and [project-card.example.md](project-card.example.md). Confirm inferred ports and backup excludes with the user when they would be destructive if wrong.

**Steps**

1. Read [commands/_placeholders.md](commands/_placeholders.md).
2. List existing `*.md` in the target commands directory. Do not overwrite a filled command unless the user asked to replace it. New files only by default; report skipped names.
3. Copy `commands/*.md` except `_*.md` into the target commands directory. Do not copy either `instantiate.md` into the product.
4. Skip `streamlit.md` when `__UI_KIND__` is `none`. Skip `rebuild.md` and `dockerfile-efficiency.md` when the product has no Dockerfile or compose file, unless the user wants stubs.
5. Fill the Project card tables and replace every `__TOKEN__`. Set `__COMMANDS_DIR__` from the platform. Fill `__BACKUP_HUB__` from the card (default `$HOME/Documents/code backups`). Grep the target commands directory for `__` and fix any leftovers.
6. Keep the standing policies intact.
7. Ensure the target `.gitignore` has the platform's line so filled copies stay local (create or append if missing). Do not ignore this repo's own `instantiate.md` files.
8. Do not modify product application code. Do not invent a new slash command in the target. Do not commit the product unless asked.
9. Report: platform, files added, files skipped, tokens filled, and any card values that were guessed.

## Edit a template (SOP)

Trigger: “update the backup template”, “normalise X”.

1. Change `commands/<name>.md` (the generic file).
2. Update [commands/_placeholders.md](commands/_placeholders.md) if you add a token.
3. Keep examples in the placeholder table generic (`myapp`). Do not add another project's name, port, or fixture path.
4. Refer to another command as `/name`. A command that runs another one carries the **Other commands** line that points at `__COMMANDS_DIR__/name.md`.

## Add a command (SOP)

A new template earns a file only when it is clearly reusable across projects. Do not add product-only runbooks (a specific pytest file list, a specific compose service) as new command files. Put those in the project card.

## Standing policies (do not weaken)

1. No `/Users/…` literals. Backup root defaults to hub `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"` with `$BACKUP_HUB=$HOME/Documents/code backups`. Sibling `"$REPO_ROOT backup"` only when the card sets `backup_hub: sibling`.
2. Mutating commands run `/backup` first. Nested backups skip when `/deep-test` or `/light-test` already ran it.
3. Artifact cleanup, `docker compose down`, and docker prune stay disabled in templates.
4. `/pre-release` is local confidence: `pass` / `warning` / `failure` / `skipped`. Never a silent pass. Never tag/push.
5. Default tests stay fast and offline.
6. Commands are plain Markdown. Filename = slash name. No YAML frontmatter: the agent is given the whole file as the prompt, and both platforms accept it that way.
7. Shell snippets use ASCII hyphens and straight quotes.
8. Secrets stay out of git.
9. The product's commands directory is gitignored. This repo tracks templates under `commands/` and the two `instantiate.md` pointers only.
10. `/backup`, `/export-exemplars`, `/streamlit`, `/rebuild`, and `/watch` are local-session commands. In a cloud or remote sandbox they report `skipped (not a local session)`. A command that needed the backup then asks the user before its first mutating step.

## Out of scope

- Migrating these commands to Agent Skills unless the user asks.
- User-global `~/.cursor/commands/` and `~/.claude/commands/` (these are project commands).
