# Project card (example)

Fill one of these when instantiating templates into a project. Paste the values into each command's **Project card** section (commands must stay self-contained — Cursor injects only the command file).

Copy this file into the consuming repo as `.cursor/project-card.md` if you want a single place to edit later; slash commands still need the values inlined.

Values below are illustrative. Replace every one. `backup_hub: sibling` is the only special case; any other hub value is a parent directory, not a `/Users/…` path.

```yaml
project: Example
package: example
src_package: src/example
ui_kind: streamlit          # streamlit | flask | none
ui_port: 8510
ui_entry: streamlit run src/example/ui/app.py --server.headless true --server.port 8510
compose_service: web
image_names: example:latest
sibling_ports:              # never inspect, bind, or kill
  - 8501
small_fixture: tests/fixtures/mini_page.png
large_fixture_hint: a small multi-page PDF under tests/fixtures/, or a path the user names
default_test_cmd: pytest -q
coverage_cmd: pytest --cov=src/example --cov-report=term-missing -q
high_leverage_tests: safety, pipeline, contracts
architecture_rules:
  - UI must not own business rules
  - core must not import Streamlit
release_governance: docs/dev/release_governance.md   # or none
doc_contracts:
  - docs/CONTRACT.md
probe_small: run the happy path on the mini fixture, offline
probe_large: run the same entrypoint on a real-sized local fixture the user names
probe_extra: smoke the UI on ui_port; skip if there is no UI
e2e_cmd: run the documented long entrypoint the user already started, or restart that same recipe
resume_hint: continue if the tool resumes; otherwise restart e2e_cmd
hang_quiet_minutes: 10
assessment_dir: assessments/
backup_hub: "$HOME/Documents/code backups"  # or sibling
backup_excludes:
  - data
  - outputs
  - .test_outputs
backup_includes: []         # extra globs; empty is fine
verify_paths:
  - src/example
  - tests
  - pyproject.toml
  - docs
  - .cursor/commands
staging_prefix: example-backup
```

See [commands/_placeholders.md](commands/_placeholders.md) for the token list and [AGENTS.md](AGENTS.md) for instantiate rules.
