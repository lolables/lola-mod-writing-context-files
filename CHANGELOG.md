# Changelog

All notable changes to this module are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-07-30

First release.

### Added

- The `writing-context-files` skill. It activates on its own trigger phrasing —
  creating, reviewing or trimming an always-loaded agent instruction file
  (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursorrules`,
  `.github/copilot-instructions.md`), asking what belongs in one, or setting up
  a repository for coding agents. Because these files are prepended to every
  turn, the skill treats authoring as subtraction and applies a fixed method:
  - the **discoverability test**, which asks whether the agent could rederive
    the line by reading the repository;
  - the **audience test**, which separates rules the agent runs from rationale
    a human reads;
  - a five-bucket **sort** of the survivors: inline, delete, link with a
    trigger, scope to a subdirectory, or annotate at the one site in the code
    the rule governs;
  - **mechanization**, proposing any rule a hook, linter or CI gate can enforce
    for migration out of the file, with the prose kept until the mechanism
    exists;
  - **validation of the cut**, handing the cut list to an independent reviewer
    to challenge, since over-cutting is the asymmetric risk.

  Audits report an `item → verdict → why` table grouped by verdict, so cuts are
  approved in batches instead of landing as a silent rewrite. The empirical
  claims rest on Gloaguen et al., *"Evaluating AGENTS.md: Are Repository-Level
  Context Files Helpful for Coding Agents?"* (2026),
  <https://arxiv.org/abs/2602.11988>.
- `references/measuring.md`, resolved from `SKILL_DIR` and read when the user
  asks whether a context file is worth keeping or how to prove it helps. Its
  audience is the human making that call, not the audit.
- Packaging as a [lola](https://lobstertrap.org/lola/) module, so `lola mod
  add` and `lola install` place the skill on any supported assistant at user or
  project scope. The module ships no `module/AGENTS.md`: lola would inject it
  into the installing assistant's context file on every turn, and nothing it
  could say passes the skill's own discoverability test.
- The quality gates behind `task check`: the structural linter (`task lint`),
  skillsaw content linting at `--strict` against a frozen baseline
  (`task lint:content`), bats suites run over deliberately-broken fixtures
  (`task test:unit`), and an install oracle that installs into a sandboxed
  `HOME` and `LOLA_HOME` at both scopes and asserts every shipped file arrived
  (`task test`). `task cleanroom` repeats the install inside a UBI10 container.
  CI runs the gates and MegaLinter; Dependabot tracks the actions.
- `docs/module-internals.md`, the maintainer-facing companion to `AGENTS.md`.
  It documents how each gate works and which of them weaken silently when
  edited, this repository's provenance from
  [lola-mod-template](https://github.com/lolables/lola-mod-template), and the
  reasoning behind the judgment calls frozen into the skill, the skillsaw
  baseline and the pinned container image.
