# Review and Expand Test Suite (/tests)

Review the `__PROJECT__` test suite for health and coverage gaps; then propose and, where safe, implement targeted test expansions.

Execute from the workspace root.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Default suite | `__DEFAULT_TEST_CMD__` |
| Coverage (optional) | `__COVERAGE_CMD__` |
| High-leverage areas | `__HIGH_LEVERAGE_TESTS__` |
| Package under test | `__SRC_PACKAGE__` |

**Other commands:** where this file says to run `/name`, read and follow `__COMMANDS_DIR__/name.md`. Do not assume one slash command can be invoked from inside another.

---

## 0. Run backup first (mandatory)

Before doing anything else, run the backup custom command (`/backup`). Wait for it to complete, then proceed.

If this `/tests` run was invoked from `/deep-test` or `/light-test` that already backed up, **skip** this step and note that in the summary.

---

## 1. Operating rules

Primary goal: improve test confidence without broad production refactors.

- Bias toward tests-only changes.
- Do not run destructive cleanup of artifacts, outputs, or user data dirs.
- If production-code changes appear necessary, stop and report the proposed fix unless it is a trivial import/path compatibility correction.
- If the default suite is failing before changes, do not expand tests until failures are understood and classified.
- Do not re-enable quarantined / skipped-by-marker tests by default.
- New tests must not be marked quarantined, slow, or `requires_*` unless explicitly justified.
- Keep default test runs **fast and offline** (no live models, no network, no real user data roots).

---

## 2. Test artifact cleanup

Cleanup is **disabled**.

Do not delete output/state/data dirs. Only report that cleanup is disabled unless the user explicitly requests a preview-only audit.

---

## 3. Review phase

### 3.1 Run and summarize current suite

If `tests/` or pytest config is missing (greenfield), report that and stop after recommending an initial smoke layout; do not invent a huge suite unprompted.

Otherwise run:

```bash
pytest --collect-only -q
__DEFAULT_TEST_CMD__
```

If debugging a failure, use `pytest -x` (or the project equivalent).

Use ASCII hyphens in shell commands (`--collect-only`, not an en-dash).

Report:

- Total collected tests
- Passed / failed / skipped / xfailed / xpassed
- Collection errors or import failures
- Whether baseline is green before adding tests

### 3.2 Structure and markers

List test directories / files under `tests/`. Inspect `pytest.ini` / `[tool.pytest.ini_options]` in `pyproject.toml`.

Confirm markers and default `addopts` when present. Do not invent a marker suite the project does not use.

### 3.3 Quarantined and skipped tests

Identify quarantined files, collection skips, and skips due to missing extras or removed APIs. Report counts and whether they should remain, be updated, or be removed. Do not re-enable unless updated to current APIs and passing.

### 3.4 Coverage and gaps

If coverage is available and reasonable, run `__COVERAGE_CMD__`. Otherwise compare `__SRC_PACKAGE__` to `tests/` by hand.

Focus expansion on `__HIGH_LEVERAGE_TESTS__`. Report which areas already have tests and which lack coverage.

---

## 4. Expansion phase

Only proceed if the baseline is green, or if baseline failures are unrelated and clearly documented.

Prefer, in order:

1. Critical paths with minimal or no tests
2. New or refactored code without tests
3. Contract tests for output shapes / invariants
4. Stable integration that does not require external dependencies

Avoid broad live end-to-end runs unless explicitly requested.

Add small, focused, deterministic, `tmp_path`-based tests. File names `test_*.py`, classes `Test*`, functions `test_*`.

Avoid absolute user paths, wall-clock timing, network, live models, and real data stores.

---

## 5. Validation

After changes, re-run `__DEFAULT_TEST_CMD__`. It must pass before finishing.

Report:

```bash
git diff --stat
git diff --name-only
```

Clearly separate production-code changes from test/doc changes.

---

## Execution summary

1. **Review** — suite status, counts, structure, skips, coverage/gaps
2. **Expansion** — files added or modified; high-leverage areas targeted
3. **Result** — final suite result, optional coverage, `git diff --stat`, any production-code changes
