# Deep Test / Harden After Plan (# deep-test)

Probe and harden after a plan has been implemented. Verify the plan landed end-to-end, deepen tests, exercise real workloads (small / large / extra), then finish with a local pre-release confidence pass.

Execute from the workspace root.

This command is **mutating when fixing issues**: fix plan gaps, test failures, and runtime errors found during probes. Prefer minimal, targeted fixes. Do not expand scope into unrelated refactors or new features.

Do not publish, push, tag, or deploy unless explicitly instructed.

## Project card (fill before first use)

| Key | Value |
| --- | --- |
| Package | `__PACKAGE__` |
| Default suite | `__DEFAULT_TEST_CMD__` |
| Small probe | `__PROBE_SMALL__` |
| Large probe | `__PROBE_LARGE__` |
| Extra probe | `__PROBE_EXTRA__` |
| Small fixture | `__SMALL_FIXTURE__` |
| Large fixture | `__LARGE_FIXTURE_HINT__` |
| Sibling ports never to touch | `__SIBLING_PORTS__` |

---

## Inputs (resolve before starting)

1. **Plan** (required): the plan just implemented — attached Cursor plan, linked plan file, or a path the user names. If none is clear, ask once, then stop.
2. **Small fixture** (default): `__SMALL_FIXTURE__`.
3. **Large workload** (default preference order): path the user names; then `__LARGE_FIXTURE_HINT__`; if none exists, stop and ask — do not invent a synthetic “large” by duplicating the mini fixture.
4. Record which mode/flags/modules were used. Prefer offline / mocked dependencies unless the plan under test requires a live service **and** that service is available.

---

## 0. Run backup first (mandatory)

Run `# backup`. Wait for it to complete, then proceed.

When later executing `# tests` and `# pre-release`, **skip their nested backup steps** if backup already succeeded in this deep-test run (note that in the summary).

---

## 1. Plan landing review (mandatory) — fix gaps

Do not treat “mostly done” as done.

For every plan phase / todo / acceptance criterion:

| Check | Action |
| --- | --- |
| Code landed | Locate symbols/files named in the plan; confirm behavior matches the written decision |
| Tests landed | Confirm planned tests exist and cover the stated cases |
| Docs landed | Confirm planned doc updates exist and match code |
| Docs + backup/restore | New features, output artifacts, or settings are documented and covered by `# backup` include/exclude |
| Explicit non-goals | Confirm out-of-scope items were not accidentally implemented |
| Contracts / schemas | Confirm versioned artifacts and invariants match the plan |

Use `git status`, `git diff`, and targeted searches. Mark each item `landed` / `partial` / `missing`.

- **Missing or partial:** implement the minimum fix to land them.
- **Drift from plan decisions:** correct code/docs/tests to match the plan (or stop and ask if the plan itself is wrong).
- Re-run focused tests for anything you change in this phase before moving on.

Do not proceed to §2 until every **required** plan item is landed or explicitly waived by the user.

---

## 2. Run `# tests` — expand and deepen (mandatory)

Execute `# tests` in full (except skip backup if already done in §0).

Emphasis on top of `# tests`:

- Prefer expansion around **code touched by the plan**.
- Keep the default suite fast/offline; do not re-enable quarantined tests without justification.
- Baseline must be green (or failures classified) before expansion; after expansion, `__DEFAULT_TEST_CMD__` must pass.

If `# tests` surfaces production bugs related to the plan, fix them, then continue.

---

## 3. Small fixture probe (mandatory)

`__PROBE_SMALL__`

Goal: prove the happy path on the tiny fixture. Watch logs; fix failures.

**Watch for:** non-zero exit, traceback, missing contracted artifacts, writes outside allowed output dirs, schema/persistence errors.

**On failure:** diagnose, fix, re-run until green (or classify as environmental skip with user confirmation).

Prefer disposable dirs under `/tmp/__PACKAGE__-deep-test-*` or the project’s documented test-output root — never into the user’s personal data tree unless they ask.

---

## 4. Large workload probe (mandatory)

`__PROBE_LARGE__`

Run on the resolved large input. Watch the terminal / logs for the entire run. Treat uncaught exceptions, non-zero exit when success was expected, and persistence errors as failures to fix.

Timebox: if hung with no progress for an unreasonable period (roughly 10+ minutes with zero log activity), kill, capture last logs, fix or classify as blocker.

---

## 5. Extra probe (mandatory when the card names one)

`__PROBE_EXTRA__`

Examples from the source projects: grouping + web UI; resume/cancel; group analysis; Streamlit smoke on `__UI_PORT__`.

If the extra surface is unavailable (no Docker, no UI extra): mark `skipped (not available)` with a **warning**; earlier steps still required.

**Never inspect, bind, or kill `__SIBLING_PORTS__`.** Stop leftover servers/processes started by this probe.

---

## 6. Run `# pre-release` (mandatory)

Execute `# pre-release` in full (except skip nested backup if §0 already succeeded).

- Treat failures related to this plan’s surface as **blockers to fix now** when safe and in-scope.
- Do not tag/push/publish.
- Carry forward probe paths into the pre-release summary if useful.

---

## Execution rules

- Work from the workspace root.
- Order is strict: §0 → §1 → §2 → §3 → §4 → §5 → §6. Do not skip ahead unless a step is impossible — then ask and wait.
- **Watch terminals** during probe steps; do not fire-and-forget long jobs.
- Prefer minimal fixes tied to plan landing or probe failures.
- Do not delete run artifacts; cleanup remains disabled.
- Do not run destructive docker prune / compose down unless the user explicitly asks.
- After any fix, re-run the smallest failing probe before continuing.

---

## Final summary (required)

1. **Plan landing** — table: landed / fixed during deep-test / waived / still open
2. **Tests (`# tests`)** — suite result; what was expanded
3. **Probes** — small / large / extra: success / skipped, paths, issues fixed
4. **Pre-release (`# pre-release`)** — `CONFIDENT` / `NEEDS FIXES` / `HIGH RISK` (not a release approval)
5. **Overall verdict** — `HARDENED` / `NEEDS FIXES` / `BLOCKED`

Also list `git diff --stat` for changes made during this run.
