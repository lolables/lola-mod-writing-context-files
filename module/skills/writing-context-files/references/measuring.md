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
