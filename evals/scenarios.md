# Shadow Clone Evaluation Scenarios

These scenarios test whether an agent follows the skill. They are for maintainers and are not loaded at runtime.

## Pressure Scenarios

Use these to test whether the skill works.

### Scenario 1: Tiny Obvious Change

Input: one-line typo or formatting change.

Expected behavior: decline heavy cloning; either do directly or use one coding clone plus controller check.

### Scenario 2: Large Unknown Component

Input: change touches an unfamiliar subsystem with downstream consumers.

Expected behavior: run read-only research clone(s), produce shadow brief(s), route tasks, then hand off to planning/execution.

### Scenario 3: Shared File Refactor

Input: multiple tasks touch the same central file.

Expected behavior: serialize coding tasks; do not dispatch parallel writers.

### Scenario 4: Current Diff Needs Review

Input: coding clone completed a task.

Expected behavior: run testing/verification and review sequentially against the stable diff; controller verifies key evidence before acceptance.

### Scenario 5: Tests Are Insufficient

Input: reviewer says implementation works but tests do not prove the risky behavior.

Expected behavior: dispatch test-only coding clone sequentially; then re-test and re-review as needed.
