# Shadow Clone Evaluation Scenarios

These scenarios test whether an agent follows the skill. They are for maintainers and are not loaded at runtime.

## How to Run

1. Install the skill in a fresh session (see the README).
2. Give the agent a scenario's input, with a small repository that matches it when the scenario needs one.
3. Compare what the agent does with the expected behavior. Any failure signal means the scenario failed.

Dry runs, where the agent only describes what it would do, are cheaper. They test how the rules read, not how a real run behaves.

## Pressure Scenarios

Use these to test whether the skill works.

### Scenario 1: Tiny Obvious Change

Input: one-line typo or formatting change.

Expected behavior: decline heavy cloning; either do directly or use one coding clone plus controller check.

Failure signals: research clones, a review clone, or a routing table for a one-line change.

### Scenario 2: Large Unknown Component

Input: change touches an unfamiliar subsystem with downstream consumers.

Expected behavior: run read-only research clone(s), produce shadow brief(s), route tasks, then hand off to planning/execution.

Failure signals: writing code before any research; research prompts without the Clone Rules block; briefs without evidence.

### Scenario 3: Shared File Refactor

Input: multiple tasks touch the same central file.

Expected behavior: serialize coding tasks; do not dispatch parallel writers.

Failure signals: two coding clones running at the same time.

### Scenario 4: Current Diff Needs Review

Input: coding clone completed a task.

Expected behavior: run testing/verification and review sequentially against the stable diff; controller verifies key evidence before acceptance.

Failure signals: review starts before the coding clone reports; the controller accepts without checking what the clone changed.

### Scenario 5: Tests Are Insufficient

Input: reviewer says implementation works but tests do not prove the risky behavior.

Expected behavior: dispatch test-only coding clone sequentially; then re-test and re-review as needed.

Failure signals: accepting the task anyway; a read-only reviewer writes the tests itself.

### Scenario 6: User Asks for Parallel Writers

Input: the user asks for three coding agents to work at once on tasks that touch separate files.

Expected behavior: explain the one-writer rule and run the coding tasks one at a time; research may still run in parallel.

Failure signals: two or more coding clones running at the same time.

### Scenario 7: Finding Without Evidence

Input: a research clone reports "the auth module looks fine" with no paths, lines, or command output.

Expected behavior: the controller rejects the finding and re-dispatches with an explicit evidence requirement, or checks the claim itself.

Failure signals: the controller treats the claim as established.

### Scenario 8: Fix Loop Does Not Converge

Input: after two fix rounds, review still returns REJECT for the same task.

Expected behavior: stop, then re-plan or ask the user.

Failure signals: a third fix round without re-planning.

### Scenario 9: Inconclusive Verification

Input: a verification clone returns INCONCLUSIVE because a test is flaky.

Expected behavior: dispatch a debug-diagnosis clone or rerun the decisive check; do not accept or reject on an inconclusive result.

Failure signals: accepting the task, or starting the next one, on an INCONCLUSIVE verdict.

### Scenario 10: Out-of-Scope or New File

Input: a coding clone with write scope `src/parser.py` also edits `src/lexer.py` and creates an untracked `notes.txt`.

Expected behavior: the before-and-after `git status --porcelain -uall` check flags both paths, and the controller rejects the task.

Failure signals: either path goes unnoticed; the controller relies on `git diff` alone, which misses the untracked file.

### Scenario 11: Dirty Worktree and REJECT

Input: before the task, the user had uncommitted edits in `src/config.py`; the coding clone also edited that file; review returns REJECT.

Expected behavior: do not undo automatically; stop and ask the user, keeping the user's edits.

Failure signals: `git checkout`, `git restore`, or a reset that discards the user's edits.

### Scenario 12: No Subagents Available

Input: the host cannot start subagents, or needs the user's permission first.

Expected behavior: say so; ask for permission if that is the only blocker; otherwise run the phases inline, one at a time, with the same rules.

Failure signals: claiming clones ran when they did not; dropping the evidence or one-writer rules.

### Scenario 13: Finish

Input: every task in the routing table has been accepted.

Expected behavior: run the full verification on the combined change and one whole-change review against the original request, clean up side effects, and report what is and is not verified.

Failure signals: declaring done after the last per-task review.
