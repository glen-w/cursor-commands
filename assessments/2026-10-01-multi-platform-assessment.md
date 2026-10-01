# Multi-platform assessment: Cursor-only, separate repo, or broaden?

Date: 2026-10-01. Repo: `glen-w/cursor-commands` on `main` at `b531f6f`, even with `origin/main`.

Question: the library was written for the Cursor IDE. Is it usable from Claude Code and other agents as it stands, does that need a separate repo, or can this one be broadened?

## Verdict

1. **As it stands, the library is not usable outside Cursor without hand edits.** Nothing loads it: the install path, the backup recipe, the gitignore rule, and `/instantiate` all name `.cursor/`.
2. **A separate repo is not warranted.** The command bodies are almost entirely platform-neutral shell procedure and policy. The Cursor coupling is about 40 lines across nine files, plus a cross-reference convention. A fork would duplicate roughly 1,300 lines of templates and every standing policy for the sake of those 40.
3. **Broaden this repo, in two phases.** Phase 1 makes the install directory a token and neutralises the wording; that covers Cursor and Claude Code with one set of files and no format change. Phase 2 adds an Agent Skills output generated at instantiate time, which covers Codex, Gemini CLI, and Copilot as well.

No template was edited for this assessment. Nothing here was tested by running the commands in another agent; the platform facts come from each vendor's current docs (see Sources).

## How each platform loads this kind of file

| Platform | Legacy command file | Skills directory it reads | Project instructions |
| --- | --- | --- | --- |
| Cursor | `.cursor/commands/<name>.md`, plain Markdown | `.agents/skills/`, `.cursor/skills/`; also `.claude/skills/` and `.codex/skills/` for compatibility | `AGENTS.md` |
| Claude Code | `.claude/commands/<name>.md`; frontmatter optional | `.claude/skills/<name>/SKILL.md` only | `CLAUDE.md`, or `AGENTS.md` when no `CLAUDE.md` exists (v2.1.277+) |
| Codex | none at project level | `.agents/skills/` | `AGENTS.md` |
| Gemini CLI | not checked | `.gemini/skills/` or `.agents/skills/` | not checked |
| GitHub Copilot | not checked | `.github/skills/`, `.claude/skills/`, `.agents/skills/` | not checked |

Three things follow from the table.

- **A plain Markdown file with no frontmatter is a valid command in both Cursor and Claude Code.** The same `backup.md` works in `.cursor/commands/` and in `.claude/commands/`. Only the destination differs.
- **Both vendors now treat command files as the older format.** Claude Code's docs say command files "still work" and to "prefer a skill for new work". Cursor ships `/migrate-to-skills` (2.4+) to convert slash commands into skills. The format this repo is built on is the legacy side on both platforms.
- **No single skills directory reaches everyone.** `.agents/skills/` covers Cursor, Codex, Gemini CLI, and Copilot but not Claude Code. `.claude/skills/` covers Claude Code, Cursor, and Copilot but not Codex or Gemini CLI. Full coverage needs both.

This repo's own `AGENTS.md` already loads in Claude Code (it was injected into this session with no `CLAUDE.md` present), so the repo-level agent instructions need no change. `/instantiate` does not: it lives only in `.cursor/commands/`, so a Claude Code user here has no `/instantiate` and relies on the trigger words in the `AGENTS.md` SOP.

## What is Cursor-specific today

### Hard coupling: breaks or misbehaves on another platform

| Where | What | Effect outside Cursor |
| --- | --- | --- |
| `README.md`, `AGENTS.md`, `instantiate.md`, `_placeholders.md`, `project-card.example.md` | Install target is `.cursor/commands/`; card lives at `.cursor/project-card.md` | Files land where no other agent looks |
| `commands/backup.md:94` | `--include='.cursor/***'` | A Claude Code project's filled commands are not in the zip |
| `commands/backup.md:129-131` | `rsync -a .cursor/commands/ "$BACKUP_ROOT/custom-commands/"` | The source directory does not exist, so rsync fails after the zip step |
| `project-card.example.md:49` | `verify_paths` lists `.cursor/commands` | Harmless when absent (the loop checks existence), but the real command directory goes unverified |
| `instantiate.md:24`, `AGENTS.md` SOP step 6 | Add `.cursor/commands/` to the product `.gitignore` | Filled copies in `.claude/` get committed, with the ports and probe recipes the policy keeps local |
| `.cursor/commands/instantiate.md` | The only installer is a Cursor command | No slash entry point from any other agent |

### Soft coupling: wording that only makes sense in Cursor

| Where | Wording | Neutral replacement |
| --- | --- | --- |
| `deep-test.md:28` | "attached Cursor plan" | "the plan from this session (plan-mode output, linked plan file, or a path the user names)" |
| `export-exemplars.md:20` | "Cursor plan for the task being implemented" | "the agent's plan for the task being implemented" |
| `export-exemplars.md:31` | "Use **semantic search**" | "Use the agent's code search (semantic when available, otherwise grep and glob)". Claude Code has no semantic index |
| `watch.md:27,30,44,111` | "the terminal file", "terminal path", "attached running terminal" | "the running process the user points at: terminal, background task, PID, or log file". Terminal files are a Cursor concept; Claude Code tracks background shell tasks |
| `backup.md:3,129` | "Mirror Cursor commands", "Back up Cursor custom commands" | "Mirror the agent command files" |
| `project-card.example.md:3` | "Cursor injects only the command file" | "the agent is given only the command file" (true of Claude Code command files too) |

### The `# name` cross-reference convention

Command files refer to each other as `# backup`, `# tests`, `# pre-release`: 34 times across the command bodies, 13 in `_placeholders.md`, 4 in `AGENTS.md`. The 2026-09-23 review flagged the `/` versus `#` mismatch (finding 3). It matters more once there is a second platform:

- `# backup` names nothing an agent can resolve. In Cursor the agent has to guess that it means `.cursor/commands/backup.md`. Nothing in the templates says so.
- In a skills install, the natural reading is "invoke the `backup` skill". That fails if the skill is marked `disable-model-invocation: true`, which is exactly the setting the mutating commands need (see Risks). The agent then has to read the file instead.

A path-explicit reference works on every platform and under both formats: "read and follow `backup.md` in `__COMMANDS_DIR__`". This is the single change that most improves cross-platform reliability, and it also closes the earlier finding.

### Branding

- The repo is named `cursor-commands`, and the README title and first line say Cursor.
- `logo.png` (and the identical `assets/logo.png`) is Cursor's cube logo, shown with `alt="Cursor"` as this project's banner. That is another company's mark on a personal repo, and it reads as official. It should go whether or not the repo is broadened.

## What is already portable

Everything else. The value of the library is in parts that no agent platform owns:

- The bash recipes: rsync and zip staging, `lsof` port handling, `docker compose build`, `pytest`, `git diff`.
- The standing policies: backup before mutation, deletes and prune disabled, sibling ports untouchable, `/pre-release` as local confidence only, no push or tag.
- The `__TOKEN__` and project-card mechanism, which is a fill step, not a platform feature.
- The self-contained-card rule. Its stated reason is that Cursor injects only the command file. Claude Code command files behave the same way, so the rule holds.

`AGENTS.md` policy 6 ("no YAML frontmatter") holds for command files on both Cursor and Claude Code. It does not hold for skills, which require `name` and `description`.

## Options

| | A. Separate repo per platform | B. Parametrise the install directory | C. Convert to Agent Skills | D. B now, skills output added at instantiate (recommended) |
| --- | --- | --- | --- | --- |
| Platforms reached | Whatever each fork targets | Cursor, Claude Code | Cursor, Claude Code, Codex, Gemini CLI, Copilot | Same as B, then same as C |
| Source of truth | Two or more copies of every body | One | One | One |
| Format change | None | None | Frontmatter on every file; directory per command | None in `commands/`; frontmatter added on install |
| Effort | High, and permanent | About 40 line edits, the cross-reference rewrite, and one new token | Restructure all 13 files, rewrite README, `AGENTS.md`, `/instantiate` | B, plus a small manifest and an `/instantiate` branch |
| Conflicts with current `AGENTS.md` | No | No | Yes: policy 6 and "do not dual-maintain as skill and command" | No: one body, wrapped on install |
| Main risk | Policy drift between forks | Stays on the format both vendors call legacy | Mutating commands become model-invocable unless flagged | Instantiate does more work; needs a leftover check for the wrapper too |

Option A fails on the repo's own terms: `AGENTS.md` exists to stop the same command being maintained twice. Option C is where both vendors are heading, but it throws away a format that works today and forces a decision on auto-invocation for every command at once. Option D gets Claude Code working with a small diff and leaves the skills move as an additive step.

## Recommendations

### Phase 1: make the existing files platform-neutral

1. **Add `__COMMANDS_DIR__` to `_placeholders.md`** with examples `.cursor/commands` and `.claude/commands`, and a matching `commands_dir` key on the example card. Use it in `backup.md` for the zip include, the default-paths sentence, and the `custom-commands/` mirror source, replacing the three `.cursor` literals.
2. **Replace every `# name` cross-reference** with a path-explicit one: "read and follow `backup.md` in `__COMMANDS_DIR__`". Keep `/backup` as the user-facing name in the README. Update `AGENTS.md` and `_placeholders.md` to match.
3. **Apply the six neutral rewordings** in the soft-coupling table above.
4. **Make `/instantiate` ask for the target platform** and derive the command directory, the card location (`.cursor/project-card.md` or `.claude/project-card.md`), and the `.gitignore` line from the answer. One platform per run; report which was used.
5. **Give `/instantiate` a second home** at `.claude/commands/instantiate.md`, or move the procedure fully into the `AGENTS.md` SOP and make both slash files two-line pointers to it. The second keeps one copy of the steps.
6. **Add a platform table to the README** (directory, file shape, gitignore line) and change the copy snippet to take the destination as a variable.
7. **Add a local-session precondition** to `/backup`, `/export-exemplars`, `/streamlit`, `/rebuild`, and `/watch`: if the agent is running in a cloud or remote sandbox, report `skipped (not a local session)` and stop. All five assume the user's own machine (`$HOME/Documents`, `~/Desktop`, loopback ports, a local Docker daemon). In a cloud agent the backup zip is written to a disk that is discarded, and every later command proceeds as if a backup exists. Cursor's background agents have this problem today; Claude Code on the web, Codex cloud, and Copilot's cloud agent add more ways to hit it.
8. **Remove the Cursor logo** from the README and delete both `logo.png` copies, or replace them with an original mark.
9. **Rename the repo** to something platform-neutral such as `agent-commands`. GitHub redirects the old URL. This is the one step that touches the remote, so it is the owner's call and can wait until the rest has landed.

After Phase 1, a Cursor user sees no behaviour change, and a Claude Code user can instantiate into `.claude/commands/` and run every command.

### Phase 2: add a skills output

Do this when a Codex, Gemini CLI, or Copilot target is actually wanted, or when Cursor stops loading `.cursor/commands/`.

1. **Keep `commands/*.md` as the only body, still without frontmatter.** Add `commands/_skills.md`: one row per command with its `description` and whether the model may invoke it unprompted.
2. **Teach `/instantiate` a skills target.** It writes `<skills-dir>/<name>/SKILL.md` as frontmatter from the manifest followed by the filled body. Skills directory is `.claude/skills/` for Claude Code and `.agents/skills/` for the rest; a user on Cursor plus Claude Code needs only `.claude/skills/`, since Cursor reads it.
3. **Amend `AGENTS.md`.** Policy 6 becomes "templates in `commands/` carry no frontmatter; the skills wrapper is generated on install". The "out of scope: migrating to Agent Skills" line and the stance against dual maintenance stay true, because there is still one body.
4. **Install one form per platform, never both.** A project with `.cursor/commands/backup.md` and `.claude/skills/backup/` shows Cursor two `/backup` entries, and they can be filled differently.
5. **Consider a shared card.** A skill directory can hold supporting files, so the 13 inlined project cards could become one `project-card.md` that each skill links to. Keep port and sibling-port values inline regardless: those are the values where a missed file read does damage.

### Not recommended

- A separate Claude repo, for the reasons under Option A.
- Converting to skills in one step. The auto-invocation risk below needs a per-command decision first.
- Platform-specific enforcement in the core templates (Claude Code `permissions.deny` rules or hooks for `docker system prune` and `compose down`). They would enforce the standing policies better than prose does, but they belong in an optional per-platform add-on, not in files that must stay portable.

## Risks to settle before a skills install

**Auto-invocation.** A command file runs only when the user types its name. A skill is, by default, also loaded by the model whenever its description matches the conversation. For this library that is a real change in blast radius: `/streamlit` kills a process, `/rebuild` rebuilds an image, `/cleanup` rewrites source files, `/deep-test` and `/watch` edit code. Each of these must carry `disable-model-invocation: true`. Claude Code and Cursor both honour that field. It is not in the Agent Skills spec's six fields, and it was not confirmed for Codex, Gemini CLI, or Copilot, so on those platforms the mutating commands may be model-invocable regardless. `/refactor` and `/pre-release` are report-only and are the only reasonable candidates for unprompted use.

**Nested calls.** `/deep-test` runs backup, tests, and pre-release in sequence. With the mutating skills hidden from the model, it cannot invoke them as skills and must read their files. This is why recommendation 2 uses file paths, and why Phase 1 should land first.

**Permission prompts.** Claude Code asks before shell commands and before writes outside the project. `/backup` writes to `$HOME/Documents/code backups` and `/export-exemplars` writes to `~/Desktop`, so a first run will prompt several times. That is the right behaviour, but the README should say so.

## Still open from the 2026-09-23 review

Checked against the current files; these are unchanged and are independent of the platform question:

- `/backup` recipe ignores `backup_hub: sibling`, and `.gitignore` is applied after the includes, so a gitignored secret under `src/` or `docs/` is zipped (finding 1).
- `/docs` mutates the tree without running backup first (finding 2).
- `/` versus `#` naming (finding 3). Recommendation 2 above resolves it.
- README says only `export-exemplars.md` has no tokens; `cleanup.md` has none either (finding 6).

Finding 1 is worth fixing in the same pass as Phase 1, since `backup.md` is being edited anyway and every other platform inherits the recipe.

## Not verified

- Cursor's current documentation page for `.cursor/commands/` could not be retrieved; the URL returned the skills page. That commands still load is taken from this repo's README and from Cursor's migration tool existing. Check before relying on Phase 1 alone for the long term.
- Whether Codex, Gemini CLI, and Copilot have an equivalent of `disable-model-invocation`.
- Gemini CLI and Copilot command-file formats and instruction-file handling were not checked; only their skills directories were.
- Whether any command name collides with a built-in on each platform. None of the 13 names matches a skill listed in this Claude Code session; other platforms were not checked.

## Sources

- Claude Code skills and command files: https://code.claude.com/docs/en/skills
- Claude Code `AGENTS.md` and `CLAUDE.md` loading: https://code.claude.com/docs/en/memory
- Cursor skills, compatibility directories, `/migrate-to-skills`: https://cursor.com/docs/context/skills
- Agent Skills specification: https://agentskills.io/specification
- Codex skills: https://learn.chatgpt.com/docs/build-skills
- Gemini CLI skills: https://geminicli.com/docs/cli/skills/
- GitHub Copilot skills: https://docs.github.com/en/copilot/concepts/agents/about-agent-skills
