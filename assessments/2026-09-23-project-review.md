# Project review and stocktake

Date: 2026-09-23. Repo: `glen-w/cursor-commands` on `main`, even with `origin/main` at `cba47eb`.

This repository is a library of plain-Markdown Cursor slash-command templates. It is not an application. Consuming projects copy `commands/*.md` (except `_*.md`) into their own `.cursor/commands/`, fill `__TOKEN__` values from a project card, and keep those filled copies gitignored. This repo stays generic.

## Working tree

Two commits:

| Commit | Subject |
| --- | --- |
| `4605337` | Add generic Cursor slash-command templates. |
| `cba47eb` | Clarify how to use the templates and add a Ko-fi link. |

Uncommitted:

- `README.md` adds a centered `logo.png` image (4 lines). The logo is not in git.
- `logo.png` is untracked (1000×1000 PNG, 136 KB). A clone of `main` has a broken image in the README until that file is committed.

No CI, no tests, no package manifest. That matches a prompt library. Nothing mechanically checks that tokens, cards, and command bodies stay aligned.

About 1,360 lines of Markdown across the README, `AGENTS.md`, the example card, `/instantiate`, and `commands/`.

## Inventory

Tracked layout matches `AGENTS.md`. Files whose names start with `_` are not slash commands. `/instantiate` lives only in this repo’s `.cursor/commands/` and must not be copied into a product.

| File | Slash | Lines | Tokens | Backup first | What it changes |
| --- | --- | --- | --- | --- | --- |
| `commands/backup.md` | `/backup` | 143 | 5 | is the backup | zip + `custom-commands/` mirror |
| `commands/tests.md` | `/tests` | 128 | 5 | mandatory; skip if `/deep-test` already did it | tests |
| `commands/deep-test.md` | `/deep-test` | 148 | 9 | mandatory | plan gaps, tests, probe fixes |
| `commands/pre-release.md` | `/pre-release` | 138 | 6 | recommended | report only |
| `commands/refactor.md` | `/refactor` | 78 | 1 | no | assessment only |
| `commands/export-exemplars.md` | `/export-exemplars` | 65 | 0 | no | copies onto `~/Desktop` |
| `commands/cleanup.md` | `/cleanup` | 77 | 0 | mandatory | format, lint, types |
| `commands/docs.md` | `/docs` | 88 | 2 | no | documentation |
| `commands/rebuild.md` | `/rebuild` | 61 | 3 | mandatory, with an internal exception | image rebuild + launch |
| `commands/streamlit.md` | `/streamlit` | 54 | 5 | no | kill and restart one port |
| `commands/dockerfile-efficiency.md` | `/dockerfile-efficiency` | 78 | 2 | mandatory | Dockerfile and `.dockerignore` |
| `commands/_placeholders.md` | do not copy | 41 | token list | — | — |
| `.cursor/commands/instantiate.md` | `/instantiate` | 33 | — | — | target repo’s `.cursor/commands/` only |
| `project-card.example.md` | — | 49 | YAML sample | — | — |

Skip rules, stated in the README and in `/instantiate`: skip `streamlit.md` when `ui_kind` is `none`; skip `rebuild.md` and `dockerfile-efficiency.md` when there is no Dockerfile or compose file, unless the user wants stubs; shrink or skip `docs.md` when the product is README-only.

## Token stocktake

`commands/_placeholders.md` lists 25 real tokens (`__NAME__` in the intro sentence is the pattern, not a token). Every real token appears in at least one command file. No command uses a token that is missing from the table. The example card’s YAML keys match those 25 tokens one-for-one.

| Token | Used in |
| --- | --- |
| `__PROJECT__` | tests, streamlit, pre-release |
| `__PACKAGE__` | deep-test, pre-release |
| `__SRC_PACKAGE__` | tests, docs, pre-release |
| `__UI_KIND__` | streamlit |
| `__UI_PORT__` | streamlit, rebuild, deep-test |
| `__UI_ENTRY__` | streamlit |
| `__COMPOSE_SERVICE__` | rebuild |
| `__IMAGE_NAMES__` | dockerfile-efficiency |
| `__SIBLING_PORTS__` | streamlit, rebuild, deep-test |
| `__SMALL_FIXTURE__` | deep-test, pre-release |
| `__LARGE_FIXTURE_HINT__` | deep-test |
| `__DEFAULT_TEST_CMD__` | tests, deep-test, pre-release |
| `__COVERAGE_CMD__` | tests |
| `__ARCHITECTURE_RULES__` | refactor |
| `__RELEASE_GOVERNANCE__` | pre-release |
| `__BACKUP_HUB__` | backup |
| `__BACKUP_EXCLUDES__` | backup, dockerfile-efficiency |
| `__BACKUP_INCLUDES__` | backup |
| `__VERIFY_PATHS__` | backup |
| `__STAGING_PREFIX__` | backup |
| `__HIGH_LEVERAGE_TESTS__` | tests |
| `__PROBE_SMALL__` | deep-test |
| `__PROBE_LARGE__` | deep-test |
| `__PROBE_EXTRA__` | deep-test |
| `__DOC_CONTRACTS__` | docs |

`export-exemplars.md` is token-free on purpose. `cleanup.md` is also token-free, and the README currently says only the export command has no tokens. Cleanup’s project card is already filled with `black`, `ruff`, and `mypy` on `src/`, `tests/`, and `scripts/*.py`. A product whose tree is not `src/` cannot fix that through the card; someone has to edit the command. `/tests` and `/docs` do follow `__SRC_PACKAGE__`. `/cleanup` does not.

`__UI_PORT__` appears in `deep-test.md` only inside an example sentence. It is not a row on that command’s project card, so a filler who only edits the card table can leave the sentinel in place. `/instantiate` does say to grep for leftover `__`.

## Policies that hold

These are consistent across `README.md`, `AGENTS.md`, `commands/_placeholders.md`, and the command bodies:

1. No `/Users/…` literals. Paths go through `$HOME`, `$BACKUP_HUB`, and `REPO_ROOT`.
2. Default backup root is `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"` with hub `$HOME/Documents/code backups`. Sibling `"$REPO_ROOT backup"` only when the card sets `backup_hub: sibling`.
3. Deletes, `docker compose down`, and docker prune stay disabled. Cleanup, rebuild, and dockerfile-efficiency all say so, and they require a preview plus an explicit ask before any delete.
4. `/pre-release` is local confidence. Per-check outcomes are `pass` / `warning` / `failure` / `skipped`. The closing line is `CONFIDENT` / `NEEDS FIXES` / `HIGH RISK`, and the command forbids version bumps, changelog edits, tags, and pushes. `/deep-test` repeats that closing line and the same ban.
5. Default tests stay fast and offline. New tests must not be quarantined, slow, or `requires_*` unless justified.
6. Commands are plain Markdown. Filename equals the slash name. No YAML frontmatter. Each command carries its own project card because Cursor injects only that file.
7. Shell snippets use ASCII hyphens. `tests.md` calls this out explicitly.
8. Secrets stay out of git and out of zips. This repo’s `.gitignore` covers `.env`, `.env.*` (with `!.env.example`), `*.pem`, `credentials.json`, and `secrets/`. It does not ignore `.cursor/commands/`, which is correct: `/instantiate` is tracked here, and filled copies are gitignored in the product.
9. `/instantiate` copies `commands/*.md` except `_*.md`, refuses to overwrite a filled product command unless asked, fills tokens, greps for leftover `__`, and does not commit the product unless asked.

## Findings

Ordered by how much they change what a filled command will actually do.

### 1. `/backup` recipe does not implement the sibling hub, and gitignore does not filter included trees

The prose in `commands/backup.md` says: if the card sets `backup_hub: sibling`, set `BACKUP_ROOT="${REPO_ROOT} backup"` and skip the hub. The bash recipe always sets `BACKUP_ROOT` from `BACKUP_HUB`. An agent that pastes the recipe will ignore the sibling override.

The same recipe lists excludes, then includes, then `--exclude='*'`, then `--exclude-from=.gitignore`. Rsync uses the first matching rule. Two consequences:

- A path under an include (`src/***`, `tests/***`, `docs/***`, `.cursor/***`, and the root globs) is kept before gitignore is consulted. A gitignored `.env` or credential file inside `src/` or `docs/` is zipped. Root `.env` is dropped by `--exclude='*'`, and the prose “always exclude” list names `.env`, `*.pyo`, `*.log`, and `*.tmp`, but those four are not in the exclude array.
- `--exclude-from=.gitignore` never sees a file the earlier rules already accepted or rejected. It does not do the job the prose assigns it. Keeping `.cursor/` even when the product gitignores it is intentional; the side effect on secrets under included trees is not.

The verify loop word-splits `__VERIFY_PATHS__` and runs an unanchored `grep` on `unzip -l`. A path with a space breaks. A shorter path can match a longer name (`src/example` inside `src/example_extra`).

Default includes are a fixed Python-shaped set: `src/`, `tests/`, `scripts/`, `docs/`, `assets/`, plus root manifests. `__BACKUP_INCLUDES__` can add globs. A tree rooted at `app/` or `lib/` is omitted until someone adds those globs. The README states the Python assumption. The recipe does not say what to add when the assumption is wrong.

### 2. `/docs` mutates the tree and does not run `/backup` first

Standing policy: mutating commands run backup first. `/docs` rewrites documentation and does not call `/backup`. `/tests`, `/cleanup`, `/deep-test`, `/rebuild`, and `/dockerfile-efficiency` do.

`/rebuild` also disagrees with itself. The greenfield paragraph says stop after an optional backup when there is no Dockerfile. Section 0 says the backup is mandatory.

### 3. `/` and `#` are both used for the same commands

The README and the public tables use `/backup`, `/tests`, `/pre-release`. `AGENTS.md`, `_placeholders.md`, and every command title and cross-reference use `# backup`, `# tests`, `# pre-release`. `export-exemplars.md` uses both in one file: the title says `# export-exemplars` and the usage line says `/export-exemplars`.

Cursor invokes these as slash commands. A filled prompt that says “run `# backup`” does not name the slash command the user actually has. Pick one spelling inside the templates and use it in the README, `AGENTS.md`, the placeholder table, and the command bodies.

### 4. Product-specific residue is still in the generic templates

`AGENTS.md` says not to merge one project’s fixture paths, ports, or runbooks back into `commands/`. These lines still read as one source project:

- `deep-test.md`: “Examples from the source projects: grouping + web UI; resume/cancel; group analysis”.
- `export-exemplars.md`: folder-name example “per-segment audio playback”.
- `streamlit.md` and `rebuild.md`: Ollama, Whisper, mail store.
- `dockerfile-efficiency.md`: “Do not bake Ollama/GGUF/vision/Whisper weights into the app image”.
- `project-card.example.md`: `tests/fixtures/mini_page.png` and “a small multi-page PDF”. The placeholder table’s generic example is `tests/fixtures/mini.json`.
- `docs.md`: a required four-layer model (CONTRACT / GUIDE / ARCHITECTURE / PRODUCT), `docs/TERMS.md`, `docs/archive/` banners, and a card cell that already names `docs/dev/docs_architecture.md`. The README says to shrink this command for a README-only repo. The command itself says that. The lint rules still fail the run when a project does not use that archive and terms layout.
- `pre-release.md`: “typical” scripts `scripts/release/assert_compose_bind.sh`, `scripts/secrets_check.sh`, `scripts/release/check_tracked_data.py`, `scripts/release/stale_refs.sh`, and `make docker-smoke`. The command says to use what exists and mark the rest skipped, so this is a softer leak than the others.

### 5. Tooling assumptions are wider than the token card

The README says the templates assume pytest, and usually ruff, black, and mypy, and that other tools mean editing the card commands. That is true for `__DEFAULT_TEST_CMD__` and `__COVERAGE_CMD__`. It is not true for:

- `/cleanup`, which hardcodes black, ruff, and mypy on `src/`.
- `/tests`, which always runs `pytest --collect-only -q` before `__DEFAULT_TEST_CMD__`. A `make test-fast` card still gets a raw pytest collect.
- `/pre-release` packaging, which assumes `python -m build` and `import __PACKAGE__`.
- `/rebuild`, which looks for `docker-compose.yml` and does not mention `compose.yaml`.

`/streamlit` is the UI command for every UI. The body says to adapt when `__UI_KIND__` is `flask`. The filename stays `streamlit.md`.

### 6. README claim about tokens is wrong

README: “`export-exemplars.md` has no tokens. The others do.” `cleanup.md` has none. Everything else in `commands/` that is a slash command has at least one.

### 7. Example card and placeholder examples disagree on purpose

Both are labelled illustrative. They still teach different shapes: port `8510` and fixture `mini_page.png` on the card, port `8501` and fixture `mini.json` in the token table; card excludes `outputs`, token examples say `output`. An instantiator who copies from both will mix them. One generic sample (`myapp`, `mini.json`, `$HOME/Documents/code backups`) should be the only sample.

## Command notes

**`/backup`.** Strongest safety intent in the set: code-only zip, no overwrite of today’s zip (falls through to `YYMMDD-HHMM`), mirror `.cursor/commands/` to `custom-commands/`, verify listed paths. The recipe gaps in finding 1 are the main defect.

**`/tests`.** Clear phase order: backup, review, expand only on a green or classified baseline, re-run the default suite, separate production diffs from test diffs. Cleanup of artifacts is disabled. Good fit for the standing rules.

**`/deep-test`.** Strict order: backup, plan landing, `/tests`, small probe, large probe, extra probe, `/pre-release`. Refuses a fake large fixture made by duplicating the mini one. Verdicts are `HARDENED` / `NEEDS FIXES` / `BLOCKED`. `__UI_PORT__` is easy to miss when filling. The “source projects” sentence should go.

**`/pre-release`.** The authority section is the clearest policy text in the repo. Script-backed checks degrade to `skipped` when the scripts are absent, which is the right default. Backup is a recommendation, and a backup failure is a warning. That matches a non-mutating report. The placeholder file still groups `/pre-release` with the nested-backup skip rule as if its backup step were mandatory.

**`/refactor`.** Assessment only. One token, `__ARCHITECTURE_RULES__`. No backup, which is correct.

**`/export-exemplars`.** Fully generic procedure: at most 10 substantive files, flat copy to `~/Desktop/<topic>/`, no tests, no docs, no padding. Topic inference when the user omits one is specified. No project card, correctly.

**`/cleanup`.** Deletes stay disabled, with a dry-run and a size abort. Backup is mandatory. Formatter, linter, and type checker are not tokens, and they ignore `__SRC_PACKAGE__`.

**`/docs`.** Useful layering rules for a project that already wants a contract tree. Heavy for everyone else, and it skips the backup rule (finding 2).

**`/rebuild`.** `compose down` and prune disabled. Build is `docker compose build --no-cache`, then launch `__COMPOSE_SERVICE__` and open `__UI_PORT__`. Sibling ports are forbidden. Backup wording contradicts itself when Docker is absent.

**`/streamlit`.** Port policy is repeated and strict: only `__UI_PORT__`, never `__SIBLING_PORTS__`. No backup. Killing a process is not a code edit; the omission is reasonable if the policy is read as “backup before changing the tree.” Worth one sentence in the command so it is an explicit exception.

**`/dockerfile-efficiency`.** Diagnosis, then Dockerfile, then `.dockerignore`, prune left disabled. Backup is mandatory. Model-weight notes are specific (finding 4) but the hygiene checklist is reusable.

**`/instantiate`.** Matches the SOP in `AGENTS.md`. It does not itself verify the backup recipe, the cleanup tool names, or the `/` versus `#` spelling.

## Verdict

The library is small, coherent, and usable. Policies on paths, prune, compose down, offline tests, and local-only pre-release are written in the same way in the README, the agent instructions, and the commands. The token table matches the commands.

The stocktake’s real gaps are operational, not missing files:

1. The `/backup` recipe can drop the sibling hub and can zip secrets that sit inside an included tree.
2. `/docs` changes the tree without a prior backup.
3. Slash names and `#` names disagree, so cross-references inside prompts do not match the command the user runs.
4. A handful of source-project examples and a non-token cleanup card still sit in files that are supposed to stay generic.
5. `logo.png` is referenced by the README and is not committed.

No template was edited for this review.
