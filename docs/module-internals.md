# Module internals

This is the maintainer-facing companion to the root `AGENTS.md`. Where
`AGENTS.md` tells an author or agent what to do, this document explains how the
scaffold that supports them works, so a future maintainer can change it safely.

## Provenance

This repository was built from
[lola-mod-template](https://github.com/lolables/lola-mod-template) by copying
its scaffold wholesale, running `task init`, and replacing the placeholder
skill with `writing-context-files` (previously a bare `SKILL.md` plus
`references/` at the repo root). Two things were then removed:

- `.taskfiles/scripts/init.sh`, the `task init` target, and the two bats tests
  covering it. `init` graduates a template into a module: it deletes the
  placeholder `example` skill and refuses to run once that skill is gone. Its
  job here is finished, its test can no longer pass, and a task that errors on
  invocation is a landmine.
- `module/AGENTS.md`. See below.
- The `task new:*` component generator and everything under it. See "Why there
  is no component generator" below.

Four smaller edits followed from those removals: `Taskfile.yml` lost the `init`
target, the `new:` include and the `NAME`/`DESC`/`KIND` vars;
`tests/verify-oracle.bats` registers the module under its real name rather than
`example`; `.skillsaw.yaml` lost the exclusion that hid the generator stubs; and
`.shellcheckrc` no longer explains itself in terms of a library that is gone.

One further deviation came later and is unrelated to the conversion: the
`Containerfile` base image and a matching `ignore` in `.github/dependabot.yml`
(see "The ubi10-minimal signature wall" below).

That is the complete deviation list. `lint-structure.sh` and
`verify-lola-module.sh` are byte-identical to the template — as their own
headers ask — and so are the fixtures, the CI workflows, the `Containerfile`'s
body below its `FROM` line, and the markdown, YAML and secret-scanning configs,
so upstream fixes can be copied straight in. `diff -rq` against a template
checkout is the cheap way to confirm that has stayed true; run it before
assuming this paragraph is still accurate.

## Why this module ships no `module/AGENTS.md`

lola injects `module/AGENTS.md` into a managed section of the installing
assistant's own context file, where it is loaded on **every turn, in every
project that installs this module**. The skill this repository ships exists to
argue that such a file must earn those tokens: keep only what the agent needs
and cannot get elsewhere.

Nothing qualified. The skill activates from the trigger phrasing in its own
`description`, which lola already registers; an injected "this module ships a
skill for context files" restates that at permanent cost. Shipping one anyway
would have been the exact failure the skill names — an inert overview that
looks like documentation.

Dropping it strands no assistant, which was the obvious risk and was checked
rather than assumed. `gemini-cli` is the one target that receives no `skills/`
directory, so it was the plausible casualty: with no `module/AGENTS.md`, would
it get anything at all? It does. A user-scope `lola install ... -a gemini-cli`
writes a `## Lola Skills` section into `GEMINI.md` listing the skill's
`description` as its trigger and pointing at the registry copy of `SKILL.md`.
That pointer is an absolute path into `.lola/`, so a `GEMINI.md` is not portable
between machines — lola's behaviour, identical for every module, not something
this repo can or should work around.

The gates accommodate this. Structural check 5 only fires when the file exists;
check 1 is satisfied by the skill. The install oracle's managed-section
assertion is guarded by `[ -e "$WORK/module/AGENTS.md" ]` and skips itself, and
installs at both scopes were verified green without it. The cost is real but
narrow: no test in this repo now exercises lola's context-file injection path.
Reintroducing the file to regain that coverage would be shipping a file to
satisfy a test, which is backwards.

## Why the root `AGENTS.md` reads the way it does

Not the `module/AGENTS.md` above — this is the repository-root file, this
repo's own always-loaded context file, and it obeys the skill it ships. Every
line in it is something a contributor's agent cannot rederive from the
repository. Layout, task names, and how the scripts work are absent because the
repository already shows them.

That constraint also explains the file's shape. It states its own scope as a
*fact* ("Layout, task names, and how the scripts work are omitted on purpose")
rather than as an instruction to go looking, because an ungated pointer at a
source the agent can already reach re-creates the tool-use cost the deletion
was meant to avoid. It reads as declarative rather than imperative prose for
the same reason, which is why `content-actionability-score` scores it low.

The file used to carry this rationale inline, in its opening paragraph. It was
moved here because the skill's own audience test puts a rule's *why* in
maintainer docs and leaves the context file with the rule alone — and a context
file arguing for its own design is the clearest possible case of prose
addressed to a human rather than to the agent that runs it. Do not put it back.

## What the A/B evaluation changed

Four passages in `SKILL.md` are the way they are because a headless A/B run said
so, not because they read well. The harness lives in `.test-output/ab/`
(untracked): two arms isolated with `CLAUDE_CONFIG_DIR` so neither inherits the
host's plugins or global context file, differing only in whether this skill is
installed; two planted fixture repositories with ground-truth keys; four
repetitions per cell; a separate judge scoring each item against the key with a
required verbatim evidence quote. Re-run it with `run-one.sh`, `judge-one.sh`
and `tally.sh` after changing the method sections.

The skill won 21.0/22 against control's 10.0/22 on auditing an existing file,
and 19.5/22 against 10.75/22 on authoring one — but the useful part is *where*.
Control scored near zero on mechanization, subtree scoping, trigger-attachment
and undefined references, and scored well on deleting waffle. A model already
knows prose is bloat; what it has no prior for is routing a rule somewhere
better than the file. That is the module's value, and it is what the method
sections must protect.

Four defects surfaced:

- **The discoverability test used to offer "a pinned toolchain version" as its
  example of something non-discoverable.** Three of four runs deleted the pin
  anyway, correctly: pins live in `go.mod`, `.nvmrc`, `pyproject.toml` — the
  first files an agent reads. The example refuted the test it illustrated. It
  now reads "a version rule no manifest already records". The same word was
  removed from the *inline* bucket and the audit checklist.
- **"Verify each named command still resolves" was a subordinate clause of the
  audit workflow's *keep* bullet**, so an item routed anywhere else skipped it.
  One run classified a `make lint-all` that does not exist as *mechanize* and
  proposed a CI gate around a missing target. Resolution is now an unconditional
  step before classification.
- **Nothing said that an unverifiable fact is not a suspect fact.** On the one
  item where the skill scored *worse* than control (0.5 against 1.5), three runs
  dropped a gotcha the user had supplied, reasoning aloud that they could not
  confirm it against the repository — "I left it out rather than guess". A fact
  with no trace in the repository is the definition of non-discoverable and the
  most valuable line available. Hence the "verification runs one way" paragraph,
  and step 5's "never silently dropped".
- **The sort had four destinations and wanted a fifth.** Asked why an agent had
  "fixed" a deliberate middleware bypass, a run proposed something the skill
  does not teach: put the constraint in a comment at the `mux.Handle` lines it
  governs. Zero per-turn cost, no drift from what it governs, read at the moment
  of editing. That is the *annotate* bucket.

Trigger behaviour needed no change and should not be casually edited: across
fifteen probes the skill fired on all direct and indirect requests, stayed
silent on "write a README" (a lazily-loaded file, which the description
excludes), and `references/measuring.md` was read in three of three runs that
asked whether the file was worth keeping and in none of the other eleven.

## What the October 2026 research refresh changed

A survey of studies, harness documentation and security advisories published
after v0.1.0 changed four things. The citations for the studies live in
`references/measuring.md`; this section records why the skill moved.

- **The cost argument.** Prompt caching bills a repeated static prefix at about
  a tenth of the input price, so "every line is paid for on every request" was
  weak as a token argument. Gloaguen et al. v3 attributes the 20–23% overhead
  to behaviour — agents follow the file and test, explore and reason more —
  and Lulla et al. measured the opposite sign (runtime −28.64%) with developer
  files on real pull requests. The intro now rests on "agents act on every
  line"; the stable finding is the absent success gain.
- **Loading behaviour is harness-specific.** The skill had assumed nearest-file
  loading and a tool-call cost for every link. Claude Code `@path` and Gemini
  CLI `@file` imports inline eagerly; Codex CLI never loads an `AGENTS.md`
  below its launch directory; Claude Code ignores `AGENTS.md` when a
  `CLAUDE.md` exists; path-scoped rule files (Claude Code `paths:`, Copilot
  `applyTo:`, Cursor `globs:`) are a scoping mechanism the sort lacked. Those
  facts rot release by release, so they live in `references/loading.md`,
  framed as questions to re-confirm, not in the method. Path-scoped rules
  joined the *scope* bucket rather than becoming a sixth: the sort section has
  no room for a sixth, and the two are the same decision — scope by place.
- **Security.** Repository-controlled agent configuration has been an
  execution vector repeatedly, and context files are an injection channel.
  GitHub advisories that motivated the security pass: GHSA-4fgq-fpq9-mr3g
  (CVE-2025-59536, Claude Code ran commands before the trust dialog),
  GHSA-mmgp-wc2j-qcv7 (CVE-2026-33068, a repository settings file bypassed the
  trust dialog), GHSA-q5hj-mxqh-vv77 (CVE-2026-40068, a spoofed worktree
  bypassed trust and ran repository hooks), GHSA-ph6w-f82w-28w6 (Claude Code
  startup-warning bypass), and GHSA-xrxf-jgv3-qmrm (CVE-2025-61260, Codex CLI
  ran repository-supplied MCP configuration). Pillar Security's "Rules File
  Backdoor" research showed invisible Unicode in rules files steering Cursor
  and Copilot, and NVIDIA's AI Red Team showed a dependency writing an
  untracked `AGENTS.md` at build time that Codex then obeyed. Hence the
  raw-byte scan, the filesystem (not git-index) inventory, and the review of
  adjacent agent config. The same advisories are why mechanization now prefers
  CI gates to agent hooks.
- **Agent-written files are the weakest kind.** Gloaguen et al. v3 found
  developer-written files beat LLM-generated ones by 7%, which strengthens the
  existing rule that a user-supplied fact with no trace in the repository is
  the most valuable line available.

The method sections changed, so the A/B harness above should be re-run before
the next release; its fixtures plant no security defect, so it measures only
that the existing method did not regress.

## What makes `module/` an installable lola module

`lola mod add` and `lola install` operate on the `module/` directory, not the
repo root. `.taskfiles/scripts/lint-structure.sh` encodes the minimum shape lola
expects:

- `module/skills/<name>/SKILL.md`: a skill, with YAML frontmatter (`name`,
  `description`) followed by its body.
- `module/agents/<name>.md`: a subagent, same frontmatter shape.
- `module/commands/<name>.md`: a slash-command wrapper.
- `module/mcps.json`: MCP server definitions, keyed under `mcpServers`.
- `module/AGENTS.md`: instructions injected for whichever assistant installs the
  module. This module deliberately ships none — see above.

Everything outside `module/` (the `.taskfiles/scripts/`, `tests/`,
`Taskfile.yml`, `Containerfile`, CI config, this doc) is repo scaffolding for
authoring and verifying the module. lola never reads it.

## Why there is no component generator

The template ships `task new:skill|new:agent|new:mcp`, which render `@@NAME@@` /
`@@DESC@@` stubs into new components. This repository deleted them, along with
their five stubs, the shared `render()`/`prompt_missing()` library, and the
twelve bats tests that covered them.

The reasoning is the one this module's own skill applies. A generator's real
value here was remembering the contract's least obvious rule — that an explicit
skill needs a command wrapper naming it — and structural check 6 already
enforces that in both directions, at build time, deterministically. A gate that
fails you beats a generator that helps you comply, and the generator cost 197
lines and a quarter of the test suite to keep green for a module that ships one
hand-written skill and produced nothing through it.

To add a component, write it: a skill is a directory under `module/skills/`
containing a `SKILL.md` whose frontmatter has `name` and `description`. Run
`task lint` and it will name anything missing. If you would rather have the
generator back, the files are unchanged in
[lola-mod-template](https://github.com/lolables/lola-mod-template) —
`.taskfiles/new.yml`, `.taskfiles/scripts/new-*.sh`,
`.taskfiles/scripts/lib/scaffold.sh`, `.taskfiles/scripts/templates/`, and
`tests/scaffold.bats` — and they were deleted unmodified, so copying them back
is a clean restore. Restore `.skillsaw.yaml`'s
`.taskfiles/scripts/templates/**` exclusion with them, or the `@@DESC@@` tokens
will fail `skillsaw --strict`.

## The nine structural invariants (`.taskfiles/scripts/lint-structure.sh`)

`task lint` enforces exactly these, and nothing more:

1. At least one installable component exists under `module/`: a skill, a
   command, an agent, `mcps.json`, or `AGENTS.md`.
2. Every skill (`module/skills/*/SKILL.md`) opens with a `---` frontmatter
   block that is also *closed* by a second `---`, and that block has both a
   `name:` and a `description:` field. An unclosed block is a failure, not a
   file with no frontmatter: everything after the opening delimiter would be
   parsed as YAML.
3. Every agent (`module/agents/*.md`) has a well-formed, closed frontmatter
   block with a `description:` field.
4. Every command (`module/commands/*.md`) has a well-formed, closed frontmatter
   block with a `description:` field. The description is what the host shows in
   its slash-command list, so a command without one is invisible in practice.
5. `module/AGENTS.md`, if present, is non-empty.
6. Explicit/wrapper pairing is bidirectional. Any skill whose frontmatter
   contains `DO NOT AUTO-INVOKE` must be named by at least one command, and
   every command must name a skill that actually exists. An orphaned wrapper is
   also an error. A command names a skill either by sharing its filename
   (`commands/<skill>.md`) or by naming it in a markdown code span — ``Invoke
   the `adr` skill via your host's Skill tool.`` — the phrasing every reference
   command uses; a bare prose mention does not count. Because the binding is by
   reference rather than by filename, one skill may be fronted by several
   differently-named commands. A module that ships commands but no skills is
   exempt from the command→skill direction: there is nothing for its commands
   to name.
7. Any skill shipping helper files beside its `SKILL.md` — `.sh`, `.py`,
   `.mjs`, `.md`, `.yml`, `.yaml`, at any depth, symlinks followed — must carry
   a `SKILL_DIR` anchor instruction in `SKILL.md` (matched loosely: `SKILL_DIR`,
   `skill-dir`, `Locate the skill`, or `Helper paths`). Without one the skill
   has no reliable way to find its own files at runtime, because the install
   destination differs per assistant and scope. A skill directory that cannot
   be fully traversed (a symlink loop, an unreadable subdirectory) warns and
   the run continues with whatever was found.
8. `mcps.json`, if present, is structurally valid: it has a `mcpServers` root
   key, and no server entry uses `"type": "remote"` (lola supports `http`/`sse`
   for remote servers; `remote` is not a real transport). This check is skipped
   with a warning if `jq` is unavailable.
9. `module/lola.yaml`, if present, declares `pre-install`/`post-install` hook
   paths that exist and resolve inside the module. An absolute path, or one
   containing `..`, is rejected without being looked up: lola refuses it too,
   and failing at lint time gives a better message than failing at install
   time.

These are cheap, offline, and don't need lola installed, so they run as the fast
first gate. Content quality (placeholder prose, weak descriptions) is skillsaw's
job, not this script's.

## The install oracle (`.taskfiles/scripts/verify-lola-module.sh`)

Structural and content lint can't catch "lola actually refuses to install
this." `task test` runs the real `lola` binary against a fully sandboxed
environment:

- `HOME` and `LOLA_HOME` are redirected into a fresh `mktemp -d` sandbox for the
  duration of the script, so a user-scope install never touches the real
  `~/.claude` or `~/.lola`.
- The module is installed from a copy of `module/` inside that same sandbox
  (`cp -R "$SRC/module" "$WORK/module"`), and `lola mod add` / `lola install`
  run with `$WORK` as the current directory. That is why a project-scope install
  (which writes into the current directory, e.g. `./.claude/`) lands inside the
  disposable sandbox instead of the repo checkout. It is also why this repo can
  safely commit a root `AGENTS.md`: nothing the oracle does writes into the real
  repo root, so the verify step can't clobber the authoring guide.
- The assertions are discovered from the source tree, not hardcoded. Before
  installing, the script enumerates `module/skills/*/` and `module/commands/*.md`
  to build the list of skills and commands it expects. After installing, it
  asserts each of them arrived somewhere under the scope root (`$HOME` for
  `--scope user`, `$WORK` for `--scope project`), that every file a skill ships
  arrived — not just its `SKILL.md` — and that lola injected the
  `lola:module:<name>` managed-section marker into a context file. Discovery
  rather than a fixed path table, for two reasons: a skill directory is named
  after the skill, not the module, so a `skills/<module-name>/` assertion is
  simply wrong for any module whose skills are named differently; and lola
  moves install destinations between releases (opencode's user-scope root moved
  from `~/.opencode` to `~/.config/opencode`), so a hardcoded table rots
  silently into a test that passes because it is looking in the wrong place.
  The searches prune the module source copy and lola's `.lola/` registry cache,
  which both contain the files being looked for and would otherwise satisfy
  every assertion vacuously. On failure the script dumps the sandbox tree to
  make a broken install debuggable.
- `--assistant` selects the target (default `claude-code`). `gemini-cli`
  receives commands and an instructions file but no skills, so the skill
  assertions are skipped for it.
- The script refuses to run against lola older than 0.5.0, the first release
  with the gitignore-aware module copy. Earlier versions copy the whole repo,
  including `.venv/` and `.git/`, which is slow and confuses the `mod add`
  walker.
- The sandbox directory is always removed on exit via a `trap`, on success or
  failure.

`task test` runs this once per scope (`user`, `project`) against the Taskfile's
`MODULE` variable, the module's identity for the gates. `task cleanroom` runs
the identical two commands again, but inside a freshly built UBI10 container, so
the oracle's result does not depend on whatever is already installed on the
host.

## The ubi10-minimal signature wall

`task cleanroom` pins `registry.access.redhat.com/ubi10-minimal:10.1` rather
than the newest tag, and Dependabot is told to leave that image alone. Both are
deliberate. The failure they avoid looks like this:

```text
Error: creating build container: unable to copy from source
docker://registry.access.redhat.com/ubi10-minimal:10.2: copying system image
from manifest list: Source image rejected: None of the signatures were
accepted, reasons: missing dev.sigstore.cosign/bundle annotation;
missing dev.sigstore.cosign/bundle annotation
```

RHEL and CentOS Stream 10 ship a default policy in
`/usr/share/containers/policy.json` (from `containers-common`) requiring
`sigstoreSigned` for `registry.access.redhat.com`, with both a `keyPath` of
`/etc/pki/sigstore/SIGSTORE-redhat-release3` and a `rekorPublicKeyPath`. The
`rekorPublicKeyPath` makes a transparency-log bundle mandatory.

What Red Hat actually publishes for `10.2` does not satisfy that:

| Tag | `dev.sigstore.cosign/bundle` | Pull under stock policy |
| --- | --- | --- |
| `10.2`, `latest` (one digest) | absent | rejected |
| `10.1` | present | succeeds |
| `10.0` | present | succeeds |

The missing Rekor bundle is not the only mismatch. Removing *just* the
`rekorPublicKeyPath` requirement in a test policy changes the error to
`cryptographic signature verification failed: crypto/rsa: verification error`,
so `10.2` is signed by a key that does not match the one the OS ships. Every
recent `10.2-<datestamp>` build behaves the same way, and `containers-common`
has no newer version carrying a key that would work. This is an upstream
mismatch between Red Hat's published signatures and Red Hat's own verification
material, not something this repository can fix.

Two traps for whoever revisits this:

- **CI cannot catch a regression here.** GitHub's `ubuntu-latest` has no
  policy for that registry, so it falls through to `insecureAcceptAnything` and
  pulls `10.2` happily. A Dependabot bump would go green and break every
  RHEL-family developer. Hence the `ignore` entry.
- **Do not "fix" it with `insecureAcceptAnything`.** It is the top search
  result for this error, and here it would not merely skip the transparency-log
  check — because the signing key does not match either, it means disabling
  signature verification for Red Hat's registry outright.

## The container-engine abstraction

`Taskfile.yml` resolves a `CONTAINER_ENGINE` variable once, at parse time:

```sh
command -v podman >/dev/null 2>&1 && echo podman || { command -v docker >/dev/null 2>&1 && echo docker || echo ""; }
```

Every `cleanroom:*` task reads `{{.CONTAINER_ENGINE}}` instead of hardcoding
`podman` or `docker`, so contributors on either engine get the same commands. A
precondition on `cleanroom:build` fails fast with a clear message if neither is
on `PATH`, before the slower build and run steps run.

## The `.skillsaw.yaml` version-pin gotcha

`task lint:content` runs `uvx skillsaw@latest lint --strict .`, always the
latest release, so a stale `uvx` cache never silently pins this repo to an
older, weaker linter. But skillsaw's ruleset version is a separate concern from
the binary version. `.skillsaw.yaml` sets `version: "0.22.0"` because if that
key is omitted, skillsaw assumes `0.6.0` and disables every rule added since. A
strict-mode run would then look green while silently skipping most of what
`--strict` is supposed to catch. Because the linter runs as `skillsaw@latest`,
keep this `version:` key in sync with whatever that currently resolves to, or
content lint quietly gets weaker instead of failing loudly.

`.skillsaw.yaml` also excludes `tests/fixtures/**` from the repo-wide
`skillsaw lint --strict .` run: those fixtures must stay structurally broken to
prove the linter catches them. The second exclusion, `.test-output/**`, covers
gitignored local scratch — the A/B harness keeps deliberately-flawed fixture
repositories there, and without the exclusion `task check` fails on any
machine that has run it while CI, which never has the directory, stays
green. `module/`
itself is deliberately not excluded. It must pass skillsaw on its own merits.

### The baselined violations (`.skillsaw-baseline.json`)

Five violations are judged, rejected, and frozen in `.skillsaw-baseline.json`.
Do not run `skillsaw fix` over this repo expecting an improvement, and do not
refresh the baseline to make a new violation go away.

`.skillsaw.yaml` promotes the rules involved from `info` to `warning`
first. That is not a claim they matter more than skillsaw thinks — it is the
only way to baseline them at all: `skillsaw baseline` captures errors and
warnings only, and silently records nothing for info-level violations no matter
what `fail-on` is set to. Left at `info`, these would be permanent `-v`
noise that a genuinely new finding could hide in.

Promoted and baselined, they are a ratchet instead. `--strict` fails on any
violation of these rules not already in the baseline, so a new bare path or an
over-long new section breaks the build. Fingerprints are content-derived, not
line-derived: the line-anchored entries survive edits that shift them up or
down the file, but rewording the flagged line itself mints a new entry.

Refresh with `uvx skillsaw@latest baseline .` after a deliberate change, and
review the diff. A shrinking baseline is good news. A growing one needs a
reason in the commit message.

Why each of the five stands:

- `content-unlinked-internal-reference` flags the genuinely-resolvable
  `docs/module-internals.md` references in the root `AGENTS.md`, and the
  `references/loading.md`, `references/measuring.md` and
  `references/security.md` pointers in `SKILL.md`, and offers to fix them. It
  should not. An agent does not click, so `[path](path)` carries no information
  a backticked path did not, at twice the tokens. Every one of these references
  already carries the thing that does matter — a trigger naming *when* to read
  it — which is what the skill this module ships argues is the payload.

Three entries left the baseline when skillsaw 0.21.0 removed
`content-actionability-score`, which had held each prose file to a floor on its
imperative-verb, command and path density. Two more left when skillsaw 0.22.0
stopped flagging paths that do not exist: `docs/grammar.md` in `SKILL.md` (a
worked example of a triggered link) and `module/AGENTS.md` in the root
`AGENTS.md` (the deliberate absence documented above).

`content-section-length` also used to be here, flagging the sort section at ~673
tokens on the argument that its four buckets are defined against each other and
could not be split. The buckets could not, but the ungated-breadcrumb passage
nested inside the *delete* bucket was never one of them — it was an aside
interrupting the comparison. Lifting it into its own `###` subsection after the
buckets took the section under the limit, reads better, and lost nothing.
The rule was right and the rejection was lazy.

It fired again when the *annotate* bucket was added, at ~521 against a 500-token
limit, and was paid for the same way: by deleting the section's one purely
explanatory line, an analogy comparing triggered links to the skill system's own
registry. The section now sits close to the ceiling with five buckets in it. A
sixth needs something else to leave.

`content-inconsistent-terminology` used to be on this list, claiming the skill
said "repository" for the thing an agent reads and "repo" only in running
prose. That was not true of the file — `SKILL.md` mixed both registers in at
least two places. Body prose now says "repository" throughout and the rule is
clean. The `description` frontmatter still says "repo" deliberately: it is the
skill's trigger surface, and it should match how users actually phrase the
request.

### The `SKILL.md` token ceiling

`SKILL.md` is about 2,943 tokens against skillsaw's 3,000-token `skill` budget —
roughly 57 tokens of headroom, two sentences. Adding to it without taking
something out will fail `task check`.

The October 2026 research refresh paid for its additions — a security-pass
trigger, a loading-reference trigger, the import-is-inlining line, path-scoped
rules in the *scope* bucket, the hook caveat — by deleting the `## Source`
section (its citations now live in `references/measuring.md`, behind the
trigger "what evidence backs this method") and collapsing two "confirm your
harness" asides into the single `references/loading.md` pointer. The next
candidate, if room is needed, is the intro's second paragraph.

`context-budget` is not one of the baselined rules. It ships at **warning**
severity, `lint:content` runs `--strict`, and warnings fail there, so this gate
goes red rather than quiet.

This is what `references/` is for. `measuring.md` was once nineteen lines, small
enough to look like pointless indirection, and rolling it into `SKILL.md` was
tried: the result was 3,227 tokens and a red gate. The reference files are
load-bearing, not decoration. Each holds content needed only at one point in the
method — the evidence and the keep-or-drop guidance for the human
(`measuring.md`), the harness loading questions asked before placing a rule
(`loading.md`), the security pass run before classifying (`security.md`) — so
each sits on the far side of a trigger naming that point.

When you hit the ceiling, pay for the addition with a deletion, or move the
least-often-needed content into `references/`. Do not raise the limit in
`.skillsaw.yaml`; the budget is the only thing keeping a skill body from growing
into a document nobody loads deliberately.

skillsaw reports a file's token count only when it exceeds the limit, so to
measure headroom, probe with a deliberately low one:

```sh
printf 'version: "0.17.0"\nrules:\n  context-budget:\n    limits:\n      skill:\n        warn: 1000\n' > /tmp/probe.yaml
uvx skillsaw@latest lint -c /tmp/probe.yaml -v .
```

## Install hooks (`module/lola.yaml`)

lola runs a script before or after installing a module, declared in
`module/lola.yaml`:

```yaml
hooks:
  pre-install: hooks/pre-install.sh
  post-install: hooks/post-install.sh
```

Paths are relative to the content directory (`module/`), and lola refuses a
path that resolves outside it. The runner invokes `bash <script>` with these
variables set:

| Variable | Value |
| --- | --- |
| `LOLA_MODULE_NAME` | the registered module name |
| `LOLA_MODULE_PATH` | the registry copy, e.g. `.lola/modules/<name>` |
| `LOLA_PROJECT_PATH` | the project directory, empty at user scope |
| `LOLA_ASSISTANT` | the target key, e.g. `claude-code` |
| `LOLA_SCOPE` | `user` or `project` |
| `LOLA_HOOK` | `pre-install` or `post-install` |

A failing `pre-install` aborts the install and removes the registry copy. A
failing `post-install` only warns — the install stands.

`lint-structure.sh` check 9 verifies that any declared hook path exists and
stays inside the module.

**Known breakage, lola v0.7.0.** Hooks do not run at user scope. The runner
calls `subprocess.run(..., cwd=project_path)`, and user scope supplies `""`.
The empty `cwd` raises `FileNotFoundError`, which lola catches and reports as
`post-install script is not executable` — a misleading message for a working,
executable script. Hooks fire correctly at project scope. Until this is fixed
upstream, a module whose default install is user-scope cannot rely on hooks for
setup work such as installing npm dependencies.

## Why install/uninstall are not Taskfile targets

`lola mod add`, `lola install`, `lola uninstall`, and `lola mod rm` are
documented directly in the README, not wrapped in `task` targets. Installing and
uninstalling a module is an end-user operation against a tool (`lola`) that
already owns scope selection, assistant targeting, and force/interactive
prompts. Wrapping it in Task would mean re-implementing or shadowing lola's own
flag surface (`-a`, `--scope`, `-f`) inside this repo's Taskfile, and that
duplication would drift the moment lola adds a flag. Task's job here is
verification (`lint`, `test`, `check`, `cleanroom`); installing the result is
lola's job.

The rule has one honest edge. A module with a genuine setup step that lola
cannot express — `docs-discipline` needs `npm install` for the mermaid linter
its skill shells out to — has nowhere to put it, because `node_modules` is in
lola's `ALWAYS_IGNORE` and never ships with an install. The right home for
that work is a `module/lola.yaml` `post-install` hook, which keeps lola in
charge of installation. Until the user-scope hook bug above is fixed, document
the extra step in the README as a prerequisite rather than wrapping `lola` in a
Task target. Wrapping is still the wrong answer; a standalone dependency target
next to the documented `lola` commands is the right one.

## Why this apparatus rather than `lola mod init`

lola ships `lola mod init`, which scaffolds `module/` with a skill, command,
agent, `mcps.json`, and `AGENTS.md`, plus a root README. If you want a component
generated rather than hand-written, that is the tool to reach for — it is
maintained upstream and costs this repository nothing.

What it does not give you is everything this repo keeps: the structural linter,
the skillsaw content gate, the sandboxed install oracle, the container
cleanroom, CI wiring, and Dependabot. `lola mod init` produces a module; this
apparatus produces a module you can prove installs. That is the reason
`.taskfiles/`, `tests/`, and `.github/` earn their maintenance cost for a
single-skill module — and the reason the generator did not.
