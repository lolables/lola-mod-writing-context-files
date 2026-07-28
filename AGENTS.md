# Working in this repository

This repository packages the `writing-context-files` skill as an installable
[lola](https://lobstertrap.org/lola/) module.

Layout, task names, and how the scripts work are omitted on purpose — the
repository shows them.

## The one rule that matters

Nothing is done until lola installs it cleanly. `task check` is the bar: it
lints structure, checks content with skillsaw, runs the bats suites, and
installs the module into an isolated sandbox at both user and project scope.

Both `task test` and `task test:unit` run the install oracle, which refuses lola
older than 0.5.0. A second, older lola under `/usr/local/bin` outranks
`~/.local/bin` unless you prepend it.

## Contract reminders

- **Invocation is either auto or explicit, never both.** Auto skills carry
  trigger phrasing in `description`; explicit skills carry a
  `DO NOT AUTO-INVOKE.` prefix AND at least one command under
  `module/commands/` that names them. One skill may be fronted by several
  commands — the linter binds them by reference, not by matching filenames.
- **Bodies are real.** No `TODO`, no placeholder prose — skillsaw `--strict`
  rejects them.
- **Install/uninstall is lola's job**, not the Taskfile's. Document the `lola`
  commands in the README; do not wrap them in tasks.
- **`mcps.json`**: stdio (`command`) or remote (`type: http|sse`). Never
  `type: remote`.
- **Assistant keys are exact:** `claude-code`, `copilot-cli`, `copilot-vscode`,
  `cursor`, `gemini-cli`, `openclaw`, `opencode`. `gemini-cli` is the one that
  gets no `skills/` directory: lola instead registers each skill by description
  inside `GEMINI.md`, pointing at the registry copy of `SKILL.md`. The install
  oracle skips its file assertions for that reason, not because the skill is
  missing.
- **This module ships no `module/AGENTS.md`, deliberately.** Do not add one back
  without a rule that passes the skill's own discoverability test; see
  `docs/module-internals.md`.
- Stage files by name, never `git add -A`.

## Before you change the scaffold

Before modifying `.taskfiles/`, `tests/`, or the CI workflows, read
`docs/module-internals.md`. It records why the gates are shaped the way they
are and which of them silently weaken if edited carelessly.
