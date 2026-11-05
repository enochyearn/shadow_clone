---
name: shadow-clone
description: Coordinates clean-context subagents, called clones, for large or risky code changes. Runs read-only research in parallel, one writing clone at a time, then fresh-context verification and review, while the main agent decides what to accept. Use for multi-file or cross-component changes, risky refactors or migrations, changes to schemas, APIs, CLIs or data contracts that other code depends on, unfamiliar code, unknown downstream consumers, or requests to delegate work or get an independent review. Skip small, obvious, or mechanical edits.
---

# Shadow Clone

## Overview

**Shadow Clone** is a controller-led discipline for using clean-context clones without turning a task into a swarm or a second build system.

Core pattern:

```text
controller stays clean
-> independent read-only clones gather evidence when needed
-> one coding clone writes one bounded task at a time
-> read-only clones test/verify/review the stable result
-> controller verifies enough evidence and decides
```

The goal is not maximum process. The goal is maximum useful intelligence per unit of context while keeping mutation single-threaded.

Shadow Clone does not replace planning, implementation, review, or verification skills. It teaches when and how to use fresh clones around those skills.

## Core Definitions

### Clone

A **clone** is a subagent started with a fresh context. It knows only the prompt the controller writes, so every rule it must follow goes into that prompt.

### Controller

The inline agent is the **controller**. It is not a clone.

The controller owns:

- user goals and constraints
- accepted plan and task order
- clone dispatch prompts
- allowed write scopes
- review and verification routing
- accumulated discoveries
- final triage and acceptance

The controller avoids absorbing unnecessary implementation detail, but still verifies enough evidence to make acceptance decisions.

### Coding Clone

A **coding clone** may edit files, but only for one bounded task.

Rules:

- Runs sequentially: one coding clone at a time, even on separate files.
- Receives a task docket, allowed write scope, acceptance criteria, and relevant discoveries.
- Writes code and/or tests only within scope.
- Reports with the Coding Clone Report: changed files, commands run, uncertainties, questions, and new discoveries.
- Does not continue into the next task.
- Does not broaden scope or architecture. If it needs to, it stops and reports the question to the controller.

Its write scope is one of:

- **implementation**: code and tests for one task
- **test-only**: tests only, used when testing or review finds a test gap
- **fix**: a bounded fix from a fix docket

### Read-Only Clone

A **read-only clone** does not edit files.

It operates in one selected mode per dispatch:

- research
- testing / verification
- review, with one or more lenses
- debug diagnosis

A read-only clone may inspect stable files, diffs, command output, artifacts, logs, tests, and documentation. It returns evidence-backed findings.

### Request, Task, Stable Revision

- The **request** is the user's whole ask. A **task** is one routed piece of it.
- A **stable revision** is the worktree after the current coding clone has stopped and reported, with no other writer active. The controller records it (see Baselines and Undo), and testing and review judge only that task's changes.

## Invariants

These rules always apply.

### 1. Independent Research Can Run in Parallel

Only independent read-only research over stable source/materials is parallel-safe by default.

Good parallel fanout:

```text
Research Clone A: map producer outputs
Research Clone B: map consumer inputs
Research Clone C: map verification surface
```

Do not parallelize research when each clone needs to judge the same changing worktree state.

### 2. Task Execution Pipeline Is Sequential

Anything that writes, runs verification against the current worktree, diagnoses current failures, or judges a specific diff happens sequentially by default.

Sequential activities:

- coding clones, in any write scope
- read-only clones verifying, reviewing, or diagnosing the current revision
- the next coding task

Do not run a testing clone while a coding clone is still editing. Do not run reviewer clones until the coding clone has stopped and reported its changed files. Do not start the next coding task until the current task is accepted (Step 5).

Multiple read-only review clones may be parallelized only when the diff is stable and the controller explicitly chooses that. Default to sequential review for clarity.

### 3. Evidence Beats Claims

Every important clone finding must be grounded in evidence:

- file paths
- line numbers when available
- diffs
- command output
- test names
- artifact paths
- explicit uncertainty

Bad:

```text
Looks good.
```

Good:

```text
`exports/foo.py` adds column `bar`, but `tests/exports/test_foo.py` only checks that the column exists. It does not assert the value semantics required by the task.
```

Before acting on a finding, the controller opens the cited `file:line` or reruns the cited command.

### 4. Fresh Context, Narrow Mission

A clone should not inherit the full chat history.

Every clone dispatch prompt must contain:

```text
role/mode
mission
bounded scope
necessary context only
constraints
clone rules
expected output
stop condition
```

The clone rules are the fixed **Clone Rules** block in `references/templates.md`. Paste it into every dispatch, because a clone cannot follow rules it never sees.

Never dispatch vague prompts like:

```text
Go understand this component.
Review this.
Figure out what to do.
```

### 5. Knowledge Flows Forward

Fresh clones forget what previous clones learned unless the controller preserves it.

After each clone that produces useful findings, the controller updates an **Accumulated Discoveries** log. Inject only relevant discoveries into future clone prompts.

Keep the log in a scratch file outside the repository, or in the plan file. Never commit it.

### 6. Check What Each Clone Changed

Before and after each clone, record `git status --porcelain -uall` plus a content hash (`git hash-object`) of each listed path, and compare the union of paths from both snapshots; the hashes catch new edits to files that were already modified. Every changed or new path must be inside the clone's write scope; for read-only clones that scope is empty, apart from controller-approved verification side effects.

Out-of-scope changes mean REJECT. Without git, compare file lists and modification times instead.

## Heuristics

These guide judgment. They are not hard rules.

### Scale Process to Risk

Size decides how to split work. Risk decides how much testing and review each task gets. Do not use every mode for every task.

Size:

```text
XS: 1 file, obvious local change
S: 1-2 files, one concern
M: 3-5 files, one feature slice
L: 6-8 files; split into M-or-smaller tasks if possible
XL: 9+ files or multiple independent subsystems; must split before implementation
```

Process by tier:

```text
XS obvious task:
  controller handles it, or one coding clone + controller check

S well-scoped task:
  coding clone -> optional review clone (one or more lenses) -> controller

M task or unfamiliar component:
  research clone(s) -> coding clone -> testing/verification -> review -> controller

High-risk interface/schema/API/data/model task, at any size:
  research clone(s) -> coding clone -> testing/verification -> interface/invariant review -> quality/spec review -> controller
```

### Decide Whether to Clone

Numbers help the controller notice when context may be getting too large, but they are not hard rules. Use them as warning lights, then make the final decision qualitatively based on risk, coupling, ambiguity, and how much judgment the task needs.

Consider using `shadow-clone` when one or more of these signals appear:

| Signal | Usually Safe Inline | Consider Shadow Clone | Strong Shadow Clone Signal |
|---|---:|---:|---:|
| Files likely touched | 1-2 | 3-5 | 6+ or shared central files |
| Files needing research | 1-3 | 4-9 | 10+ |
| Lines of existing code to inspect | <300 | 300-999 | 1,000+ |
| Largest relevant file | <300 lines | 300-799 lines | 800+ lines |
| Components involved | 1 | 2 | 3+ |
| Downstream consumers | none/known | 1-2 | 3+ or unknown |
| Verification commands | 0-1 simple test | 2-3 targeted checks | smoke/canary/artifact inspection |
| Risky interfaces changed | none | one local interface | schema/API/CLI/artifact/model/data contract |

Qualitative factors can override the numbers:

- A one-file change can still need `shadow-clone` if it touches a schema, API, security boundary, generated artifact, model feature, migration, or shared identity/key logic.
- A many-file change may not need heavy cloning if it is mechanical, low-risk, and already covered by clear tests.
- Unknown consumers, unclear invariants, surprising test gaps, or disagreement between agents are stronger signals than raw file count.
- If coordinating clones would take longer than understanding the task directly, skip cloning.

Typical fits: M-sized or larger tasks, or high-risk small ones; large repos whose local detail would pollute the controller's context; producer/consumer boundaries; implementation that should start from a fresh, narrow context; focused inspection of tests, logs, or artifacts; and work that deserves a fresh-context review.

Then ask:

```text
Do I understand the component boundaries?
Do I know what inputs/outputs/interfaces must be preserved?
Do I know which tests or commands prove success?
```

If any answer is no, use research clones.

```text
Could a fresh reviewer catch things my current context will miss?
Is the task too broad for one clean implementation prompt?
```

If either answer is yes, use a review clone or split the task.

Do not use `shadow-clone` when:

- the task is tiny and obvious
- the relevant context fits comfortably in the inline session
- the change is mechanical and low-risk
- the task is already broken into safe executable steps with adequate review
- coordination overhead would exceed the benefit

Never use it to run more than one writer at a time.

## Relationship to Existing Skills

Shadow Clone should complement, not replace, existing skills. The skills below are separate and not included here. Use them if installed; otherwise use this skill's templates.

| Need | Prefer, if installed |
|---|---|
| Break a spec into tasks | `planning-and-task-breakdown` |
| Write a detailed implementation plan | `writing-plans` |
| Execute a plan task-by-task | `subagent-driven-development` |
| Analyze huge code/docs/search spaces | `recursive-decomposition` |
| Bundle evidence for independent review | delegated review / review-bundle style workflows |

Shadow Clone adds the missing bridge:

```text
big messy reality
-> bounded evidence-backed shadow briefs
-> task routing and review lenses
-> clean handoff to planning/execution/review skills
```

When a formal implementation plan already exists, do not duplicate it. Use Shadow Clone only to prepare missing evidence, routing, review lenses, or accumulated discoveries. After a handoff, the one-writer, evidence, change-check, and finish rules still apply.

## Process

### Step 1: Decide Whether to Clone

Apply **Decide Whether to Clone** above. If clones would not help, work inline.

### Step 2: Bound the Work

Define one or more slices. A slice can be:

- component
- producer surface
- consumer surface
- integration surface
- verification surface
- current diff
- specific failure

Good slice:

```text
Map how the export layer produces dataset X and which downstream components consume it.
```

Bad slice:

```text
Understand the whole repo.
```

### Step 3: Run Read-Only Research Clones When Needed

Research clones gather evidence before implementation.

They must:

- search before reading broadly
- stay inside the assigned boundary
- identify entrypoints, inputs, outputs, consumers, invariants, and verification
- mark uncertainty
- return a compact brief

### Step 4: Controller Synthesizes and Routes

The controller turns clone findings into:

- task list
- dependency order
- task sizes
- allowed write scopes
- review lenses
- verification commands
- handoff packet: the routing table plus relevant discoveries, for the next skill
- accumulated discoveries

If the work is not yet executable, hand off to `planning-and-task-breakdown` or `writing-plans` if installed, or write the plan with the Task Routing Table.

If the work is executable, hand off to `subagent-driven-development` if installed, or dispatch the next bounded coding clone.

### Step 5: Run the Per-Task Pipeline

For each task, work through this checklist. Testing and review steps apply when the task's tier requires them.

```text
- [ ] Record the baseline (Baselines and Undo)
- [ ] Dispatch one coding clone with a task docket
- [ ] Wait until it stops and reports changed files
- [ ] Check what it changed against its write scope
- [ ] Testing/verification clone, if the tier requires it
- [ ] Review clone with selected lenses, if the tier requires it
- [ ] Controller spot-checks key evidence and decides (Outcomes)
- [ ] Fix rounds as needed, each followed by re-test / re-review: at most 2, then stop and re-plan or ask the user
- [ ] Update discoveries
- [ ] Accept the task, then start the next one
```

### Step 6: Finish

After the last task is accepted:

1. Run the full verification on the combined change, not only the per-task checks.
2. Run one whole-change review against the original request, focused on interfaces between tasks.
3. Clean up verification side effects that are not part of the deliverable.
4. Report to the user what changed, what is verified (with evidence), what is not, and open risks.

### Baselines and Undo

- Before each coding task, record the starting point: a commit if the user allows commits and the tree has no uncommitted user edits, otherwise the status-and-hash snapshot plus a saved diff. Never sweep the user's pre-existing edits into a baseline commit.
- On REJECT, undo automatically only if the task's files had no uncommitted user changes when the task began. Otherwise stop and ask the user; never discard work that existed before the task.

### Outcomes

| Outcome | Next step |
|---|---|
| Verification FAIL, cause clear | Dispatch a coding clone with a bounded fix docket |
| FAIL with unclear cause, or INCONCLUSIVE | Dispatch a debug-diagnosis clone, then a fix clone; or rerun the decisive check yourself |
| Review REJECT | Undo the task's changes per Baselines and Undo (ask first if the user had uncommitted edits in those files), then redo it with a revised docket or re-plan |
| ACCEPT WITH MINOR FIXES | Apply or dispatch a small fix, then rerun only the affected checks |
| Clone dies, gives no report, or ignores the format | Retry once with a narrower mission; then do the step inline or ask the user |
| Still failing after 2 fix rounds | Stop; re-plan or ask the user |
| Clones disagree | Check the cited evidence yourself; if still unclear, dispatch one tie-break clone with both claims or ask the user; record the decision under Decisions Made |

### When Clones Are Not Available

If the host has no subagent tool, needs the user's explicit permission to start subagents, or the controller is itself a subagent that cannot start more: say so, and ask for permission if that is the only blocker. Otherwise run the same phases inline, one at a time, with the same templates, one-writer rule, and evidence rule.

When the host offers a read-only agent type or sandbox, use it for read-only clones. Still run the change check, because some hosts remove only edit tools and shell commands can still write.

## Read-Only Clone Modes

### Research Mode

Purpose: Build a reliable shadow model of one bounded slice before implementation.

Research mode returns a **Shadow Research Brief**.

### Testing / Verification Mode

Purpose: Assess what a stable implementation result actually proves.

A testing/verification clone answers:

- What commands were run?
- What passed or failed?
- What do these tests actually prove?
- What do they not prove?
- Are outputs, artifacts, snapshots, generated files, or canaries consistent with the task?
- Are there missing edge cases or coverage gaps?

It does not edit files. If tests must be written or changed, dispatch a sequential coding clone with test-only write scope.

Verification commands may write caches, artifacts, snapshots, generated files, or external state. Run only controller-approved safe commands, and report any observed side effects.

### Review Mode

Purpose: Inspect a stable result from a clean context.

The controller selects one or more review lenses:

| Lens | Checks |
|---|---|
| Spec compliance | Task requirements, acceptance criteria, scope creep, user constraints |
| Interface / invariant | Inputs, outputs, schemas, APIs, CLIs, files, artifacts, consumers, rules that must remain true |
| Code quality | Structure, naming, maintainability, consistency, error handling, duplication, side effects |
| Test adequacy | Whether tests prove the right behavior, missing edge cases, weak assertions |
| Verification | Whether command output/artifacts support the success claim |

### Debug Diagnosis Mode

Purpose: Explain a specific failure and propose the smallest fix path.

A debug diagnosis clone is read-only and returns a Debug Diagnosis Report. If the fix should be applied, dispatch a coding clone with a bounded fix docket.

## Templates

All templates, including the Clone Rules block, are in `references/templates.md`. Copy the Clone Rules block and the matching template into each clone prompt.

## Anti-Patterns

| Anti-pattern | Looks like | Fix |
|---|---|---|
| Runaway Clone Tree | Clones spawning clones without controller checkpoints. | Every clone reports back to the controller. The controller decides the next clone. The Clone Rules block forbids clones from starting other agents. |
| Context Dump | Giving a clone the whole chat history or entire repo context. | Give narrow mission, relevant excerpts, and explicit scope. |
| Parallel Writers | Two coding clones run at the same time, even on separate files, or make design decisions in parallel. | One writer at a time. Split tasks and serialize. |
| Stale Testing | Testing clone runs while code is still changing. | Testing/verification starts only after coding clone stops and reports changed files. |
| Rubber-Stamp Review | Reviewer says "looks good" without grounded findings or verdict. | Reject generic praise. Require evidence and explicit verdict. |
| Bureaucratic Pipeline | Every task gets every clone mode. | Scale review/testing to risk. |
| Shadow Theory Instead of Shadow Evidence | Clone returns plausible architecture guesses without evidence. | Require paths, commands, diffs, outputs, and confidence labels. |
