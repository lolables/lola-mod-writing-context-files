# How harnesses load context files

Every placement decision assumes a loading behaviour, and harnesses differ. For
each harness the repository targets, confirm these points from its current
documentation — not memory — before relying on a bucket. The examples record
documented behaviour as of October 2026; they change between releases.

When a repository targets several harnesses, place each rule where every target
loads it, or name the harness that will miss it.

## What loads unconditionally

In most harnesses the root file, and ancestors up to the launch directory, load
at session start. Subdirectory files and rule files often load only when the
agent touches matching paths. Only the unconditional set carries the
per-session cost — and only it is certain to survive context compaction: Claude
Code re-injects the root file afterwards, while nested files and path-scoped
rules return only when a matching file is read again.

## Imports splice content in

An import directive inlines its target at load time — Claude Code `@path`,
Gemini CLI `@file` — so it costs exactly what inlining costs. Only a plain link
or path mention is read on demand. Never use an import to "link"
sometimes-needed content.

## Subdirectory files

Claude Code, Gemini CLI, Cursor and the GitHub Copilot agents pick up a nested
file when work touches its subtree. Codex CLI concatenates files from the
repository root down to the launch directory and never loads one below it: a
`backend/AGENTS.md` is invisible to a Codex session started at the root.

## Path-scoped rules

Rule files with glob frontmatter load only when matching files are in play —
Claude Code `.claude/rules/*.md` (`paths:`), Copilot
`.github/instructions/*.instructions.md` (`applyTo:`), Cursor `.cursor/rules/`
(`globs:`). This scopes by file pattern rather than directory, at no tool-call
cost; prefer it to a nested file when the rule follows a file type rather than a
folder.

## Which filename wins

Claude Code reads `AGENTS.md` only when no `CLAUDE.md` exists at or above the
working directory, unless configured otherwise. OpenCode prefers `AGENTS.md`
over `CLAUDE.md`. The Copilot coding agent reads `CLAUDE.md` and `GEMINI.md`
only at the repository root. A repository carrying both files can have one
silently ignored: keep one source and import or symlink it from the other.

## Size caps

Codex CLI stops concatenating at `project_doc_max_bytes` (32 KiB by default),
dropping the nearest — most specific — files first. Claude Code caps on-demand
files by context window (40,000 characters at 200K). Other harnesses publish
guidance, not truncation.
