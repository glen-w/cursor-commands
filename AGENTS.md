# AGENTS.md

Instructions for agents working in this template library. It is prompts, not a product. Do not invent a product, stack, or biography here.

## Read order

1. [README.md](README.md)
2. This file
3. [commands/_placeholders.md](commands/_placeholders.md)
4. Then the task

## What this repo is

Project-agnostic Cursor slash commands. Consuming projects keep **filled** copies under their own `.cursor/commands/`. This repo stays generic. Do not merge product-specific pytest file lists, fixture paths, or ports back into `commands/` unless they belong in the project-card tokens.

## Stance

Push back on:

- A new command that is a one-off (one project, one sitting)
- Re-enabling `# cleanup` deletes, `docker compose down`, or docker prune inside a template
- Hard-coded `/Users/…` paths
- Treating `# pre-release` as the tag gate
- Dual-maintaining the same command as both a skill and a slash command here (slash commands stay `.md` under `commands/`)

## Layout

```
commands/                 # templates to copy → <project>/.cursor/commands/
  _placeholders.md        # token list; do not copy
  *.md                    # one file per slash command; filename = /name
.cursor/commands/         # commands for *this* repo only
  instantiate.md
project-card.example.md
LICENSE
```

Files whose names start with `_` are not slash commands.

## Instantiate (SOP)

Trigger: “instantiate”, “copy commands into X”, `/instantiate`.

1. Target must be a git project the user names.
2. Copy `commands/*.md` except `_*.md`. Do not copy `.cursor/commands/instantiate.md` into the product.
3. Do not overwrite filled product commands unless asked.
4. Fill `__TOKEN__` values from the product card. Grep for leftover `__`.
5. Skip UI/Docker commands when the product has no UI/Docker, unless the user wants stubs.
6. Ensure the target `.gitignore` lists `.cursor/commands/` so filled copies stay local. Do not ignore this repo's `.cursor/commands/instantiate.md`.
7. Do not commit the product unless asked.

## Edit a template (SOP)

Trigger: “update the backup template”, “normalise X”.

1. Change `commands/<name>.md` (the generic file).
2. Update [commands/_placeholders.md](commands/_placeholders.md) if you add a token.
3. Keep examples in the placeholder table generic (`myapp`). Do not add another project's name, port, or fixture path.

## Add a command (SOP)

A new template earns a file only when it is clearly reusable across projects. Do not add product-only runbooks (a specific pytest file list, a specific compose service) as new command files. Put those in the project card.

## Standing policies (do not weaken)

1. No `/Users/…` literals. Backup root defaults to hub `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"` with `$BACKUP_HUB=$HOME/Documents/code backups`. Sibling `"$REPO_ROOT backup"` only when the card sets `backup_hub: sibling`.
2. Mutating commands run `# backup` first. Nested backups skip when `# deep-test` already ran it.
3. Artifact cleanup, `docker compose down`, and docker prune stay disabled in templates.
4. `# pre-release` is local confidence: `pass` / `warning` / `failure` / `skipped`. Never a silent pass. Never tag/push.
5. Default tests stay fast and offline.
6. Commands are plain Markdown. Filename = slash name. No YAML frontmatter (Cursor injects the whole file as the prompt).
7. Shell snippets use ASCII hyphens and straight quotes.
8. Secrets stay out of git.
9. Product `.cursor/commands/` is gitignored. This repo tracks templates under `commands/` and `instantiate.md` only.

## Out of scope

- Migrating these commands to Agent Skills unless the user asks.
- User-global `~/.cursor/commands/` (these are project commands).
