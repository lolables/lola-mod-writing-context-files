# writing-context-files

A [lola](https://lobstertrap.org/lola/) module shipping one skill,
`writing-context-files`, that writes or audits the always-loaded agent
instruction files: `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.cursorrules`,
`.github/copilot-instructions.md`, and friends.

---

> 🤖 LLM WARNING 🤖
>
> This project was written with LLM (AI) assistance.
>
> 🤖 LLM WARNING 🤖

---

## What the skill does

These files are prepended to every agent turn, so every line is paid for on
every request. The skill treats authoring as subtraction and applies a fixed
method:

- The _discoverability test_ asks whether the agent can rederive a line by
  reading the repository. If it can, cut the line. Architecture overviews and
  "how it works" prose are the archetype: widely recommended, and measurably
  inert.
- The _audience test_ asks who the line speaks to, the agent that runs the file
  or a human reading it. Rationale, history, and justification belong in a
  commit message or `docs/`. Keep the rule, cut the essay around it.
- The _sort_ puts every survivor in one of five places. Inlined (always
  needed), deleted (never needed), linked _with a trigger_ (sometimes needed),
  scoped to a subdirectory (needed only there), or annotated as a comment at
  the one site in the code it governs.
- _Mechanization_ outranks prose. Any rule a hook, linter, or CI gate can
  enforce is a candidate for moving out of the file, because deterministic beats
  probabilistic. The prose stays until the mechanism exists.
- _Validating the cut_ comes last. Over-cutting is the asymmetric risk, so the
  cut list goes to an independent reviewer and gets challenged adversarially
  before it stands.

Audits produce an `item → verdict → why` table grouped by verdict, so you can
approve in batches rather than accept a silent rewrite.

The skill activates on its own trigger phrasing. It fires when you create,
review, trim or "clean up" a context file, ask what belongs in one, complain
that one is too long or ignored, or set up a new repo for coding agents. There
is no slash command; it is an auto-invoked skill.

The empirical claims it rests on come from Gloaguen, Mündler, Müller, Raychev,
Vechev, _"Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for
Coding Agents?"_ (2026), <https://arxiv.org/abs/2602.11988>.

## Prerequisites

- lola, the module installer:

  ```sh
  uv tool install git+https://github.com/LobsterTrap/lola@v0.7.0
  ```

  See <https://lobstertrap.org/lola/> for what lola is and how it works.
- [uv](https://docs.astral.sh/uv/) provides the `uv`/`uvx` used above and by
  `task lint:content`.
- [task](https://taskfile.dev) runs every gate in this repo (`task check`,
  `task lint`, `task test`). Needed to develop the module, not to use it.

## Installation

Register the module, then install it. The interactive form prompts for
assistant and scope:

```sh
lola mod add -n writing-context-files ./
lola install writing-context-files
```

Scripted, non-interactive:

```sh
lola mod add -n writing-context-files ./
lola install writing-context-files -a claude-code --scope user -f
```

## Uninstall

```sh
lola uninstall writing-context-files
lola mod rm -f writing-context-files
```

## Quality gates

`task check` runs everything a reviewer will expect to be green:

1. `task lint` is the structural linter
   (`.taskfiles/scripts/lint-structure.sh`): frontmatter, descriptions,
   explicit/wrapper pairing, the `SKILL_DIR` anchor a skill needs when it ships
   helper files, and so on.
2. `task lint:content` hands the module to skillsaw
   (`skillsaw@latest lint --strict`). It catches placeholder prose, malformed
   descriptions, and other authoring mistakes structural checks can't see.
3. `task test:unit` covers the structural linter (against deliberately-broken
   fixtures) and the install oracle with bats. It needs `bats` and `lola` on
   `PATH`: several of its tests perform real sandboxed installs.
4. `task test` is the install oracle
   (`.taskfiles/scripts/verify-lola-module.sh`). It runs `lola mod add` and
   `lola install` against a sandboxed `HOME` and `LOLA_HOME`, at both user and
   project scope, then asserts every file the skill ships actually arrived
   (`references/measuring.md` included).

`task cleanroom` repeats the install-verify step inside a fresh UBI10 container
(podman preferred, docker fallback), so "works on my machine" isn't a hidden
dependency.

The module is not done until `task check` is green.

## Repository layout

| Path                                   | Role                                                                |
| -------------------------------------- | ------------------------------------------------------------------- |
| `module/`                              | The installable module. lola reads only this.                       |
| `module/skills/writing-context-files/` | The skill: `SKILL.md` plus `references/measuring.md`.               |
| `.taskfiles/`                          | The two quality-gate scripts: structural linter and install oracle. |
| `tests/`                               | bats suites and the deliberately-broken fixtures they lint.         |
| `docs/module-internals.md`             | Maintainer documentation for the gates.                             |

To grow the module, add the component by hand under `module/` (a skill is a
directory holding a `SKILL.md` with `name` and `description` frontmatter), then
let `task lint` tell you what the contract requires. An explicit skill carries a
`DO NOT AUTO-INVOKE.` prefix and needs a command under `module/commands/` that
names it; the linter enforces that pairing in both directions. `AGENTS.md` has
the full contract.

## See also

- [lola-mod-template](https://github.com/lolables/lola-mod-template), the
  template this repository was built from.
- [lolafy](https://github.com/lolables/lola-mod-lolafy) converts an existing
  skill from another host into a lola module.
- lola itself: <https://lobstertrap.org/lola/>

## License

This module is licensed under the [MIT License](LICENSE). lola is a separate
project with its own license; installing lola, or this module, does not bring
lola's license into your project.
