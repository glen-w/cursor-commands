# Placeholders

Do not copy this file into `.cursor/commands/`. Tokens are `__NAME__` so they grep cleanly. After instantiate, **zero** `__…__` tokens should remain in a project command.

| Token | Meaning | Examples |
| --- | --- | --- |
| `__PROJECT__` | Display name | My App |
| `__PACKAGE__` | Import / dist name | `myapp` |
| `__SRC_PACKAGE__` | On-disk package | `src/myapp` |
| `__UI_KIND__` | UI runtime | `streamlit`, `flask`, `none` |
| `__UI_PORT__` | Loopback port | `8501` |
| `__UI_ENTRY__` | Start command | `streamlit run src/myapp/ui/app.py --server.port 8501` |
| `__COMPOSE_SERVICE__` | Compose service `# rebuild` launches | `web`, `ui` |
| `__IMAGE_NAMES__` | Image tags `# dockerfile-efficiency` diagnoses | `myapp:latest` |
| `__SIBLING_PORTS__` | Ports belonging to other local projects | another local UI on `8502` when this project uses `8501` |
| `__SMALL_FIXTURE__` | Canonical tiny fixture | `tests/fixtures/mini.json` |
| `__LARGE_FIXTURE_HINT__` | How to pick a real-ish workload | a path the user names under `fixtures/` |
| `__DEFAULT_TEST_CMD__` | Fast offline suite | `python -m pytest tests/ -q`, `pytest -q`, `make test-fast` |
| `__COVERAGE_CMD__` | Optional coverage | include `--cov=` for `__SRC_PACKAGE__` |
| `__ARCHITECTURE_RULES__` | Intended boundaries for `# refactor` | UI must not own business rules |
| `__RELEASE_GOVERNANCE__` | Authoritative next-tag doc, or `none` | `docs/dev/release_governance.md` |
| `__BACKUP_HUB__` | parent dir for all project backups (no `/Users/…`) | `$HOME/Documents/code backups` |
| `__BACKUP_EXCLUDES__` | rsync `--exclude` names (no leading slash) | `data`, `output`, `fixtures`, `.test_outputs` |
| `__BACKUP_INCLUDES__` | extra include globs beyond the default set | `archive/***`, `*.lock` |
| `__VERIFY_PATHS__` | paths that must appear in the zip when they exist | `src/myapp`, `tests`, `pyproject.toml` |
| `__STAGING_PREFIX__` | mktemp prefix | `myapp-backup` |
| `__HIGH_LEVERAGE_TESTS__` | areas `# tests` should expand first | safety, pipeline, contracts |
| `__PROBE_SMALL__` | `# deep-test` happy-path recipe | run the documented entrypoint on the mini fixture, offline |
| `__PROBE_LARGE__` | `# deep-test` heavier recipe | same entrypoint on a real-sized local fixture the user names |
| `__PROBE_EXTRA__` | third probe | web UI, resume, or another surface named on the card |
| `__DOC_CONTRACTS__` | authoritative contract files for `# docs` | `docs/CONTRACT.md` |

Standing policies that are **not** tokens (do not weaken them per project):

1. No hard-coded home directories (`/Users/…`). Use `$HOME` / `$BACKUP_HUB`.
2. Backup root defaults to `"$BACKUP_HUB/$(basename "$REPO_ROOT") backup"` with `__BACKUP_HUB__` = `$HOME/Documents/code backups`. Sibling `"$REPO_ROOT backup"` only when the card sets `backup_hub: sibling`.
3. Mutating commands run `# backup` first. Nested backups inside `# tests` / `# pre-release` are skipped when `# deep-test` already backed up.
4. Artifact cleanup, `docker compose down`, and docker prune stay **disabled** unless the user names a small, previewed set.
5. `# pre-release` is local confidence, not the tag gate. Outcomes are `pass` / `warning` / `failure` / `skipped`. Never a silent pass.
6. Default tests stay fast and offline.
7. Do not publish, push, or tag unless the user explicitly asks.
