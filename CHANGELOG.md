# Changelog

All notable changes to this module are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this project adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- A security pass, run on every audit and authored file, inline in `SKILL.md`
  with its commands and checklist in `references/security.md`. It treats a context file the user did not write as untrusted input:
  inventory every instruction file the harness would load, including untracked
  ones and symlinks leaving the repository; scan raw bytes for invisible and
  bidirectional Unicode and list HTML comments and `<details>` blocks from the
  file on disk, since a harness may strip them from the copy it loads; flag
  instructions that exfiltrate data, follow remote content, install from
  unverified sources or weaken safety; propose removing secrets; and review
  the agent config beside the file (hooks, MCP servers, auto-approve settings,
  folder-open tasks, endpoint overrides) that executes code.
  Findings lead the audit table under a new `security` verdict.
- `references/loading.md`, the loading questions to confirm per target harness
  before placing a rule: eager `@` imports, which subdirectory files load (Codex
  CLI never loads one below its launch directory), path-scoped rule files,
  which filename wins when several exist, and size caps.

### Changed

- The authoring workflow reads `references/loading.md` before placing rules
  and ends with the security pass. An A/B re-run found authoring agents never
  reached either trigger while it sat only in the intro and audit paragraph.
- The *scope* bucket now covers path-scoped rule files (glob-matched rules)
  alongside subdirectory files, and the *link* bucket states that an import
  directive is inlining, not linking.
- Mechanization prefers CI gates and linters to agent-harness hooks: a
  committed hook is code every clone runs with its user's privileges, and a
  model-judged hook is probabilistic again.
- The cost framing no longer rests on per-turn token price, which prompt
  caching makes small; it rests on agents acting on every line.
- `SKILL.md` was audited with its own method to make room for the inline
  security pass: 358 tokens of rationale and restatement removed after an
  adversarial review, every instruction kept.
- The evidence moved from the skill body to `references/measuring.md` and now
  cites Gloaguen et al. v3 (developer-written files beat LLM-generated ones by
  7%), Lulla et al. (an `AGENTS.md` cut runtime 28.64%, so the cost effect is
  setup-dependent), Khatri (no correctness effect; the one useful line was a
  non-discoverable gotcha) and McMillan (file size and structure did not affect
  adherence).
- `.skillsaw.yaml` tracks skillsaw 0.22.0, drops the removed
  `content-actionability-score` rule, and excludes the gitignored
  `.test-output/` scratch directory. The baseline went from eight entries to
  seven: five old ones resolved, and the new `references/` trigger pointers are
  rejected for the reason already recorded.

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
