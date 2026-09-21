# Workspace Cleanup & Quality Pass (# cleanup)

Run a code quality and hygiene pass to ensure the codebase is clean, consistent, and type-safe.
Execute from the workspace root.

After running, summarize issues found and confirm whether the workspace is clean.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Format | `black src/ tests/ scripts/*.py` (or the project formatter) |
| Lint | `ruff check src/ tests/ --fix` |
| Types | `mypy src/` |
| Safe scratch dirs (preview-only) | project output / test-output dirs — never repo root or `src/` |

---

## 0. Run backup first (mandatory)

Run `# backup`. Wait for it to complete, then proceed.

Skip if `# deep-test` already backed up in this run.

---

## Dry-run first & sanity checks (mandatory)

- **Dry-run first:** Any destructive step must have a preview/list step run first; apply/delete only after reviewing.
- **No recursive delete at root:** No `rmtree`/`rm -rf` on `.`, `..`, the workspace root, `src/`, or any dir containing `pyproject.toml`. If a command would delete outside a known-safe scratch subdir, do not run the apply step and report the risk.
- **Flag or abort on large deletions:** List or estimate size/count first. If the scope suggests a huge deletion (e.g. > ~1 GB or a very large file count), **do not delete**; report and skip. Only remove a small, explicitly confirmed set after the user agrees.
- **Never delete user project/data dirs** without explicit per-path confirmation.

---

## 0b. Cleanup before quality pass

Delete/remove steps stay **disabled** after repeated data loss. Re-enable only with explicit user request and extreme care.

- **List (dry-run only)** ad hoc summaries and test reports in `tests/` without deleting (`TEST_*_SUMMARY.md`, `TEST_*_REPORT.md`, `TEST_*_ASSESSMENT.md`). Report the list; do **not** delete.
- If the user asks to clean artifacts, list candidates under the project’s output / test-output dirs and only delete with explicit confirmation.

---

## 1. Formatting

- Run the project formatter on `src/`, `tests/`, and `scripts/*.py` (config in `pyproject.toml` when present).
- Do not run `black .` at repo root (can touch `.venv` / site-packages).
- After formatting, run the formatter in check mode to confirm no further changes.

---

## 2. Linting

- `ruff check src/ tests/ --fix` (or the configured linter).
- Re-run if auto-fixes landed.
- List remaining warnings or errors that are not auto-fixable.

---

## 3. Type checking

- `mypy src/` (or project roots as in `[tool.mypy]`). Use `--ignore-missing-imports` if the project does so.
- List file and line for each error.
- Propose only the minimal change to satisfy the type checker. Do not refactor beyond that unless explicitly requested.

---

## Execution Rules

- Do **not** introduce new features.
- Do **not** perform structural refactors.
- Only fix **formatting**, **lint**, and **type** issues unless explicitly asked.
- After completion, provide a short summary with:
  - **Formatting:** what was changed (files/lines) or "no changes."
  - **Lint:** issues fixed and any remaining warnings/errors.
  - **Types:** type errors found and minimal fixes applied or suggested.
