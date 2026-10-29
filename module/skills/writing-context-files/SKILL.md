---
name: writing-context-files
description: >-
  Write or audit repository context files — AGENTS.md, CLAUDE.md, GEMINI.md,
  .cursorrules, .github/copilot-instructions.md, or any always-loaded agent
  instruction file. Use this whenever the user wants to create, review, trim,
  or "clean up" one of these files, asks what belongs in AGENTS.md/CLAUDE.md,
  complains a context file is too long or ignored, or is setting up a new repo
  for coding agents — even if they don't name this skill. Not for lazily-loaded
  docs (plain READMEs, skill bodies, on-demand `docs/` pages) whose per-turn cost
  is near zero.
---

# Writing context files

Repository context files (AGENTS.md, CLAUDE.md, and friends) load into
**every** agent session, and agents act on every line — measured, they do not
reliably raise success, and often raise cost. Your job is subtraction: keep
only what the agent both needs and cannot get elsewhere.

This reasoning is premised on *always-loaded* files; for a lazily-loaded skill
body or `docs/` page it is far weaker. Harnesses differ in what they load and
when: before placing or auditing anything, read `references/loading.md`
(relative to this skill's directory, `SKILL_DIR`).

The file itself is optional: if the tests below empty it, a near-empty file or
none is the right output — don't pad.

When the user asks whether a file is worth keeping, how to prove it helps, or
what evidence backs this method, read `references/measuring.md` — that
guidance is for the human making the call, not for the audit itself.

## The discoverability test

For each candidate line, run the **discoverability test** — it decides most of
the file:

> **Can the agent discover this by reading the repository itself?**

- **No** (non-discoverable) → it might earn its place. Examples: the project's
  wrapped test command when it sets flags or spins up fixtures a raw invocation
  skips, a version rule no manifest already records, a convention that
  contradicts the language default, a gotcha with no trace in the code ("CI runs
  in UTC; tests assume it").
- **Yes** (discoverable) → cut it. The agent reconstructs architecture,
  module layout, and "how it works" from source faster than it trusts a prose
  summary — and the summary rots. This is the category measured as
  **inert**: repository overviews, despite being widely recommended, buy
  nothing while costing tokens on every turn.

Verification runs one way: checking a line against the repository tells you
whether it is discoverable, never whether it is true. A fact the user gives you
that leaves no trace there is not suspect — that absence is what makes it
non-discoverable. If it looks doubtful, ask; never resolve doubt by deleting.

Distrust the "but it's expensive to reconstruct across N files" plea for keeping
a map or overview. In practice a single source of truth — a registry, a
`main()`, a pipeline definition — almost always collapses that cost and makes the
map an overview after all. Find that file before believing the expense; the
genuinely irreducible cross-file map is far rarer than it feels.

Shortness is a *consequence* of applying this test, not the goal. Don't trim a
genuinely non-discoverable convention to save lines; do delete a whole
"Architecture" section even if it's beautifully written.

## The audience test

The **discoverability test** catches repository content the agent can rederive.
A second, broader filter — the **audience test** — catches what it can't: **is this
line addressed to the agent that runs the file, or to a human reading it?** These files are consumed per turn by an LLM,
not read once by a person — so a rule's *rationale* ("we do this because…"), its
history, and any case for why it matters are human-facing and belong in a commit
message or `docs/`; the agent needs the rule, not the argument for it. Keep the
instruction, cut the essay around it.

The positive form of both tests is one question: **name the concrete task an
agent gets *wrong* without this line.** If you can't, it's decoration — whoever
it's for.

## Inline / delete / link / scope / annotate — the sort

Every survivor lands in exactly one bucket, decided by *how often* and *where*
it's needed:

**Always needed → inline it.** Linking always-relevant content is strictly
worse than inlining: you pay a tool-call round-trip on top of loading the same
tokens. Exact build/test commands and hard constraints live here.

**Never needed → delete it.** The discoverable-overview case. If it's valuable
to humans, it belongs in maintainer docs (a separate `docs/` file the agent
opens on demand), not in the always-loaded context file.

**Sometimes needed → link it, *with a trigger*.** The only band where a pointer
beats both inlining and deletion. But a bare `see also: docs/grammar.md` is the
worst option — it costs a tool call and gives the agent no basis to decide
whether following it is worth the cost. Attach the condition:

> Before modifying the parser, read `docs/grammar.md`.

The **trigger — the "when" — is the payload**, not the path. The agent can grep
its way to the file; what it cannot infer is *when that file becomes relevant*,
and that "when" is itself non-discoverable — which is what justifies the
deferred load. An import directive that splices the file in at load time is
inlining, not linking.

**Needed only in a subtree or for some files → scope it, don't centralize.** A
rule governing only `/backend` belongs in `/backend/AGENTS.md`, or a path-scoped
rule file, not the root — centralizing it taxes every turn spent elsewhere.
Scoping is lazy-loading by *place*: when a rule maps cleanly onto a directory or
file pattern it beats even a triggered link, costing no tool call.

**Tied to one site in the code → annotate it there.** A constraint governing a
specific function, route table or config block can live as a comment on that
line: no per-turn cost, no drift from what it governs, and it is read at the
moment of editing. Useless for a rule with no single site.

### The ungated-breadcrumb trap

When you delete discoverable content, resist leaving a breadcrumb that tells the
agent to go fetch it — "read the `git log` for history," "see the repository for
architecture." Aimed at something the agent already knows how to reach, that is
an ungated pointer at an unbounded source: it re-creates the tool-use cost you
deleted the content to avoid. State scope as a *fact* ("this file omits
architecture; the repository shows it"), never as an *instruction to fetch*. A
trigger-gated link to a *specific* doc is different — that is the *sometimes
needed* case above.

## Prefer mechanization over prose

A rule written in prose is *probabilistically* followed — the agent might skip
it deep in a long turn. The same rule as a **hook, lint rule, pre-commit
check, or CI gate** is *deterministically* enforced. Whenever a rule is
mechanizable, moving it out of the file upgrades the guarantee from "usually"
to "always" and shrinks the context tax to zero.

- "Never commit `.env`" → a pre-commit hook, not a sentence.
- "Route test artifacts to `.test-output/`" → the build script that writes them there.
- "Run the linter before pushing" → a CI gate.

Leave a rule in prose only when it genuinely can't be mechanized (judgment
calls, style-of-reasoning guidance). Automation shrinks the set; the
discoverability test then filters whatever residue remains.

Mechanization is a *proposal*, not work you perform or force on the user. Name
the mechanism by kind — a secret-scanning pre-commit hook, an import-boundary
linter, a CI gate — and if you name a concrete tool, choose one that fits the
repository's actual stack, not a default. Prefer CI gates and linters to
agent-harness hooks: a committed hook is code every clone runs with its user's
privileges, and one that asks a model to judge is probabilistic again. And
sequence it safely: **the prose stays until the mechanism exists.** Cutting "never commit secrets" in anticipation of a
hook nobody has added drops the rule rather than moving it. Present mechanization
as a migration the user adopts at their pace; only a rule whose gate is already
in place leaves the file now.

## Workflow

**Authoring a new file:**

1. List everything you're tempted to include.
2. Run the discoverability test on each item; drop every "yes."
3. Route mechanizable rules to hooks / linters / CI gates; note them for the user.
4. Place each survivor: inline (always), linked-with-trigger (sometimes),
   scoped to a subtree (local to one directory), or annotated at its site in
   the code.
5. Write it tight — every line non-discoverable and imperative — and account
   for every item from step 1: placed, mechanized or deleted, never silently
   dropped.

**Auditing an existing file:** before classifying anything, resolve every
command, target and path the file names against the repository. An item routed
to *mechanize* or *scope* never gets checked otherwise, and proposing a CI gate
around a `make` target that no longer exists automates a footgun — agents obey
wrong instructions as readily as right ones. Run the security pass in
`references/security.md` — on authoring too. Then classify each section:

- Overview / architecture / "how it works" prose → flag for **deletion**
  (offer to relocate to maintainer docs).
- Commands and counter-default conventions → **keep**.
- Bare "see also" links, or a named rule/convention the file references but never defines → attach a trigger (or the definition), else delete. An unresolved pointer costs the agent a search or a wrong guess, whichever the name is.
- Enforceable rules sitting in prose → propose mechanizing them.

Don't silently rewrite a file the user has invested in — surface the reasoning
and let them decide; see Output for the report format.

## Validate the cut, then loop

The asymmetric risk here is *over-cutting*. Keeping an inert line wastes a few
tokens; deleting a genuinely non-discoverable convention is a silent loss that
only surfaces later, when an agent gets something wrong it had no way to know.
So verify the deletions, not the survivors, and prefer an independent pass to
your own re-read — the author is worst-placed to spot their own over-confidence.

1. Hand the *cut list* — everything marked delete or relocate — to a fresh
   reviewer that has not seen your reasoning. Frame it adversarially: for each
   item, argue that an agent *could not* rederive it from the repository.
   Anything that survives the challenge was misclassified — restore it.
2. Reconcile honestly. A survived challenge outranks the urge to cut.
3. Loop until a pass surfaces nothing new, or twice — whichever comes first.

A large or high-stakes file earns the loop; a thirty-line one does not. With no
reviewer available, re-read the cut list cold against the repository, asking the
discoverability question of each line as if you had never written it.

## Output

When auditing, produce one table — columns `item → verdict → why` — grouped by
verdict (security / delete / keep / mechanize / scope / annotate /
fix-trigger) so the user can approve in batches. Split rows at the granularity of a single *decision*,
not a whole section. A mostly-discoverable section routinely hides one keeper —
a branch-naming convention buried in a git-worktree tutorial, a tenant-isolation
*exception* inside a protobuf block. Give that keeper its own row (e.g. `carve
from §N → keep`); a coarse "delete this section" verdict sweeps it away with the
discoverable bulk around it, and the loss is silent. When authoring, produce the
file, a brief list of the rules you moved to automation instead of prose, and
any security findings.
Keep the meta-commentary out of the context file itself — it, too, must pass the
discoverability test.
