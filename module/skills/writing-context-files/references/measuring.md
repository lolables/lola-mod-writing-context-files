# Is a context file worth keeping? (guidance for the human deciding)

You almost never need a rigorous A/B. A full task-success benchmark (run N real
tasks with the file and without it, compare success rates) burns far more compute
than the file could ever save, so it is the wrong default. Use cheap proxies
instead, in order:

1. **Name the failure.** For each line, name the concrete task an agent would get
   *wrong* without it. If you can't, the line is suspect. This is the
   discoverability and audience tests applied statically, and it decides almost
   everything at zero cost.

2. **Ablate, sparingly.** For the rare high-stakes line that survives step 1 but
   you still doubt, drop that one section and run three or four representative
   tasks with and without it. Change one thing at a time, or the signal is noise.

Reserve real measurement for genuine marginal calls. The static tests in the
skill body handle the rest — and if step 1 empties the file, that is a finding,
not a failure: a near-empty file or none is a legitimate result.

## Evidence

- Gloaguen et al., *"Evaluating AGENTS.md: Are Repository-Level Context Files
  Helpful for Coding Agents?"* (v3, September 2026),
  <https://arxiv.org/abs/2602.11988>. Context files do not significantly change
  task success, and raise inference cost by 20–23% because agents follow them —
  more testing, exploration and reasoning. Instructions are well followed;
  repository overviews are not helpful. Developer-written files beat
  LLM-generated ones by 7%, so a file an agent drafts from what it read is the
  weakest kind.
- Lulla et al., *"On the Impact of AGENTS.md Files on the Efficiency of AI
  Coding Agents"* (2026), <https://arxiv.org/abs/2601.20404>. With developer
  files on real pull requests, median runtime fell 28.64% and output tokens
  16.58%. The cost effect depends on the file and setup; the absence of a
  success gain is the more stable finding.
- Khatri, *"Do Context Files Help Coding Agents? A Two-Agent Ablation Study on
  Real Repositories"* (2026), <https://arxiv.org/abs/2607.27250>. No measurable
  correctness effect across Claude Code and Codex; agents failed on
  implementation skill, not missing repository knowledge. The one line that
  changed behaviour was a non-discoverable gotcha: warned that the full test
  suite is slow, the agent ran targeted tests instead.
- McMillan, *"Instruction Adherence in Coding Agent Configuration Files"*
  (2026), <https://arxiv.org/abs/2605.10039>. File size, instruction position,
  file architecture and contradictions showed no detectable effect on
  adherence; compliance decayed within a session as the agent generated more
  code. Length is a cost argument, not an adherence one — and
  rules that must hold late in a long session are the ones to mechanize.
