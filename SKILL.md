---
name: shadow-clone
description: Use when large, risky, cross-cutting, or context-heavy work needs clean-context subagents while preserving controller judgment and avoiding parallel writers.
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

- Runs sequentially.
- Receives a task docket, allowed write scope, acceptance criteria, and relevant discoveries.
- Writes code and/or tests only within scope.
- Reports changed files, commands run, uncertainties, and new discoveries.
- Does not continue into the next task.
- Does not broaden architecture without asking the controller.

If tests need to be added or changed, dispatch a coding clone with a **test-only write scope**.

### Read-Only Clone

A **read-only clone** does not edit files.

It can operate in one selected mode:

- research
- testing / verification
- review
- debug diagnosis

A read-only clone may inspect stable files, diffs, command output, artifacts, logs, tests, and documentation. It returns evidence-backed findings.

## Hard Rules

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

- coding clones
- test-writing clones
- testing / verification clones against the current diff
- reviewer clones judging the current diff
- debugger clones diagnosing current failures
- fix clones
- the next coding task

Default task pipeline:

```text
Controller
-> Coding Clone for Task N
-> Read-Only Clone in testing/verification mode
-> Read-Only Clone in review mode
-> Controller judgment
-> Fix Coding Clone if needed
-> re-test / re-review if needed
-> accept Task N
-> Coding Clone for Task N+1
```

Do not run a testing clone while a coding clone is still editing. Do not run reviewer clones until the coding clone has stopped and reported its changed files. Do not start the next coding task until testing, review, controller judgment, and required fixes for the current task are complete.

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

### 4. Fresh Context, Narrow Mission

A clone should not inherit the full chat history.

Every clone dispatch prompt must contain:

```text
role/mode
mission
bounded scope
necessary context only
constraints
expected output
stop condition
```

Never dispatch vague prompts like:

```text
Go understand this component.
Review this.
Figure out what to do.
```

### 5. Knowledge Flows Forward

Fresh clones forget what previous clones learned unless the controller preserves it.

After each clone that produces useful findings, the controller updates an **Accumulated Discoveries** log. Inject only relevant discoveries into future clone prompts.

### 6. Scale Process to Risk

Do not use every mode for every task.

```text
XS obvious task:
  controller handles it, or one coding clone + controller check

S well-scoped task:
  coding clone -> optional combined review clone -> controller

M task or unfamiliar component:
  research clone(s) -> coding clone -> testing/verification -> review -> controller

High-risk interface/schema/API/data/model task:
  research clone(s) -> coding clone -> testing/verification -> interface/invariant review -> quality/spec review -> controller
```

### 7. Use Numeric Signals as Heuristics

Numbers help the controller notice when context may be getting too large, but they are not hard rules. Use them as warning lights, then make the final decision qualitatively based on risk, coupling, ambiguity, and how much judgment the task needs.

Consider using `shadow-clone` when one or more of these signals appear:

| Signal | Usually Safe Inline | Consider Shadow Clone | Strong Shadow Clone Signal |
|---|---:|---:|---:|
| Files likely touched | 1-2 | 3-5 | 6+ or shared central files |
| Files needing research | 1-3 | 4-10 | 10+ |
| Lines of existing code to inspect | <300 | 300-1,000 | 1,000+ |
| Largest relevant file | <300 lines | 300-800 lines | 800+ lines |
| Components involved | 1 | 2 | 3+ |
| Downstream consumers | none/known | 1-2 | 3+ or unknown |
| Verification commands | 0-1 simple test | 2-3 targeted checks | smoke/canary/artifact inspection |
| Risky interfaces changed | none | one local interface | schema/API/CLI/artifact/model/data contract |

Qualitative factors can override the numbers:

- A one-file change can still need `shadow-clone` if it touches a schema, API, security boundary, generated artifact, model feature, migration, or shared identity/key logic.
- A many-file change may not need heavy cloning if it is mechanical, low-risk, and already covered by clear tests.
- Unknown consumers, unclear invariants, surprising test gaps, or disagreement between agents are stronger signals than raw file count.
- If coordinating clones would take longer than understanding the task directly, skip cloning.

## When to Use

Use `shadow-clone` when:

- the repo is large enough that inline context will get polluted by local details
- the task touches multiple components or producer/consumer boundaries
- you need to understand interfaces, schemas, commands, consumers, or invariants before writing code
- implementation should happen from a fresh, narrow coding context
- review should happen from a fresh context rather than the implementer context
- tests/logs/artifacts need focused inspection
- the task is M-sized or larger, or high-risk despite being small
- hidden downstream consumers, generated artifacts, or verification gaps could matter

Do not use `shadow-clone` when:

- the task is tiny and obvious
- the relevant context fits comfortably in the inline session
- the change is mechanical and low-risk
- the task is already broken into safe executable steps with adequate review
- coordination overhead would exceed the benefit
- you are tempted to spawn parallel writers over overlapping files

## Relationship to Existing Skills

Shadow Clone should complement, not replace, existing skills.

| Need | Prefer |
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

When a formal implementation plan already exists, do not duplicate it. Use Shadow Clone only to prepare missing evidence, routing, review lenses, or accumulated discoveries.

## Process

### Step 1: Decide Whether to Clone

Ask:

```text
Do I understand the component boundaries?
Do I know what inputs/outputs/interfaces must be preserved?
Do I know which tests or commands prove success?
Could a fresh reviewer catch things my current context will miss?
Is the task too broad for one clean implementation prompt?
```

If no, use one or more shadow clones.

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
- handoff packet
- accumulated discoveries

If the work is not yet executable, hand off to `planning-and-task-breakdown` or `writing-plans`.

If the work is executable, hand off to `subagent-driven-development` or dispatch the next bounded coding clone.

### Step 5: Run Sequential Task Pipeline

For each task:

1. Dispatch one coding clone.
2. Wait until it stops and reports changed files.
3. Dispatch testing/verification clone if verification needs focused inspection.
4. Dispatch reviewer clone with selected lenses.
5. Controller verifies key evidence and decides.
6. If fixes are needed, dispatch a bounded coding clone.
7. Re-test / re-review as needed.
8. Update discoveries.
9. Move to next task only after acceptance.

## Read-Only Clone Modes

### Research Mode

Purpose:

```text
Build a reliable shadow model of one bounded slice before implementation.
```

Research mode returns a **Shadow Research Brief**.

### Testing / Verification Mode

Purpose:

```text
Assess what a stable implementation result actually proves.
```

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

Purpose:

```text
Inspect a stable result from a clean context.
```

The controller selects one or more review lenses:

| Lens | Checks |
|---|---|
| Spec compliance | Task requirements, acceptance criteria, scope creep, user constraints |
| Interface / invariant | Inputs, outputs, schemas, APIs, CLIs, files, artifacts, consumers, rules that must remain true |
| Code quality | Structure, naming, maintainability, consistency, error handling, duplication, side effects |
| Test adequacy | Whether tests prove the right behavior, missing edge cases, weak assertions |
| Verification | Whether command output/artifacts support the success claim |

### Debug Diagnosis Mode

Purpose:

```text
Explain a specific failure and propose the smallest fix path.
```

A debug diagnosis clone is read-only. If the fix should be applied, dispatch a coding clone with a bounded fix docket.

## Templates

Output templates and the clone dispatch prompt template are in `references/templates.md`. Copy the matching template into each clone prompt.

## Anti-Patterns

### Runaway Clone Tree

Clones spawning clones without controller checkpoints.

Fix: every clone reports back to the controller. The controller decides the next clone.

### Context Dump

Giving a clone the whole chat history or entire repo context.

Fix: give narrow mission, relevant excerpts, and explicit scope.

### Parallel Writers

Two coding clones edit overlapping files or make design decisions in parallel.

Fix: one writer at a time. Split tasks or serialize.

### Stale Testing

Testing clone runs while code is still changing.

Fix: testing/verification starts only after coding clone stops and reports changed files.

### Rubber-Stamp Review

Reviewer says "looks good" without grounded findings or verdict.

Fix: reject generic praise. Require evidence and explicit verdict.

### Bureaucratic Pipeline

Every task gets every clone mode.

Fix: scale review/testing to risk.

### Shadow Theory Instead of Shadow Evidence

Clone returns plausible architecture guesses without evidence.

Fix: require paths, commands, diffs, outputs, and confidence labels.

## Summary

Shadow Clone is a discipline:

```text
Research before risky changes.
One writer at a time.
Fresh eyes for review.
Testing/review happens against stable results.
Evidence for everything.
Knowledge carries forward.
Controller decides.
```
