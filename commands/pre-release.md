# Pre-Release Check (# pre-release)

Local **developer-confidence** report for `__PROJECT__`. Execute from the workspace root.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Authoritative next-tag doc | `__RELEASE_GOVERNANCE__` |
| Default test command | `__DEFAULT_TEST_CMD__` |
| Small fixture (output sanity) | `__SMALL_FIXTURE__` |
| Package | `__PACKAGE__` / `__SRC_PACKAGE__` |

---

## Authority (hard rules)

This command is **not** the release gate and **must not**:

- Change the package version
- Edit `CHANGELOG.md`
- Create, inspect, or approve git tags for release
- Push commits
- Recommend a push or tag
- Claim to block, verify, or create tags
- Present itself as authoritative release governance

**Authoritative next-tag checklist:** `__RELEASE_GOVERNANCE__`. If that is `none` or the file does not exist, say so and keep this report local-confidence only.

**Outcomes for every check:** `pass` / `warning` / `failure` / `skipped` (with reason). Environment-dependent checks that cannot run report **`skipped`**, never a silent pass.

Do not assume PyPI upload unless the project actually publishes there. Docker checks only when Docker packaging exists.

---

## Shared scripts (prefer these over ad-hoc commands)

When present under `scripts/release/` or `scripts/`, prefer them. If missing, run the equivalent inline checks below and mark script-backed rows **`skipped` (script not present)**.

Typical names (use what exists; do not invent):

| Check | Script (when present) |
| --- | --- |
| Compose loopback bind | `make docker-smoke` / `bash scripts/release/assert_compose_bind.sh` |
| Denylist / secrets | `bash scripts/secrets_check.sh` |
| Tracked data allowlist | `python3 scripts/release/check_tracked_data.py` |
| Stale refs + TODO gate | `bash scripts/release/stale_refs.sh` |

---

## 0. Optional backup

If the user has not already run `# backup` in this session, recommend running it. Backup failure is a **warning** for this local-confidence command (not a tag authority).

Skip if `# deep-test` already backed up in this run.

---

## 1. Worktree snapshot

- Report `git status --short` / porcelain v1 with `--untracked-files=all` when git is initialized.
- Dirty or unexpected paths → **failure** for local readiness (do not advise tagging).
- No git repo yet → **warning** (greenfield).
- Branch / remote sync are informational; being behind remote → **warning**.

---

## 2. Tests (when locally available)

- Interpreter gate: `python --version` must satisfy `requires-python` in `pyproject.toml`.
- Run `__DEFAULT_TEST_CMD__` (and smoke/contract Makefile targets when present).
- Failures → **failure**. Unavailable pytest/env → **skipped** with reason.
- Do not require live models or network for the default suite.

---

## 3. Packaging smoke

- `python -m build` when packaging is set up (install `build` if needed).
- Install wheel into a throwaway venv when practical; confirm `import __PACKAGE__` and version.
- Failure → **failure**. Missing packaging config → **skipped**.
- Inspect sdist/wheel: user data, `.venv`, caches, and secrets must not ship. Unexpectedly large artifacts → **failure**.

---

## 4. Compose + Docker (optional)

- Only if `docker-compose.yml` / `Dockerfile` exists.
- Run bind/smoke scripts when present (source-level checks do not need a daemon).
- When Docker is available: a documented image smoke if one exists.
- Docker unavailable or no packaging → **skipped** — never pretend pass.
- Do not run `docker compose down` or prune.

---

## 5. Hygiene gates

- Run secrets / tracked-data / stale-ref scripts when present.
- Spot-check that data dirs, large binaries, and `.env` are not tracked.
- Any denylist / secrets / tracked-data failure → **failure** for the local readiness report. **Do not** recommend tagging or pushing.

---

## 6. Docs + backup/restore coverage

Compare this branch / dirty worktree against the base branch (or the session’s plan) for **new or changed features, output artifacts, and settings**.

| Check | Pass when | Failure / warning |
| --- | --- | --- |
| Documented | Each new surface appears in the right doc layer | Undocumented user-visible knob → **failure**; minor index drift → **warning** |
| `# backup` paths | Code/config/docs included; generated/runtime dirs excluded | New top-level code/doc path missing from backup includes → **failure** |
| Restore / reopen | Persisted settings still load; commands restorable from `"$BACKUP_ROOT/custom-commands/"` (the hub or sibling root `# backup` just used) | Broken reopen → **failure** |

No new surfaces in the diff → **pass** (N/A).

---

## 7. Soft / optional

- `black --check` / `ruff check` / `mypy` → report; auto-fix only if the user asks (this command is non-mutating by default).
- Docs drift beyond §6 → **warning**.
- CI status via `gh` when available → informational; missing CI/gh → **skipped**.
- Output sanity on `__SMALL_FIXTURE__` when the project has a cheap golden-path command (digest, analysis, OCR). Missing fixture → **skipped**.

---

## Final summary

| Area | Outcome |
| --- | --- |
| Worktree | pass / failure / warning |
| Tests | pass / failure / skipped |
| Packaging | pass / failure / skipped |
| Docker | pass / failure / skipped |
| Hygiene | pass / failure / skipped |
| Docs + backup/restore | pass / failure / warning / skipped |

Then list failures and warnings. End with a **local confidence** line: `CONFIDENT` / `NEEDS FIXES` / `HIGH RISK` — explicitly **not** a release approval.
