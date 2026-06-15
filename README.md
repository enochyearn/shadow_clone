# Shadow Clone

When building complex software with AI agents, unstructured swarms fail. Parallel agents step on each other's toes, hallucinate contracts, and make conflicting design choices.

The better pattern is:

```text
many readers
one writer
controller decides
check, don't guess
```

The `shadow-clone` skill is how you gather intelligence and use agents to write code safely.

## The Concept

Inspired by the Naruto technique, a Shadow Clone is a temporary agent spawned with a completely clean context.

A clone has one focused mission. It might scout a bounded component, map an interface, review a stable diff, verify command output, diagnose a failure, or implement one scoped task.

When its mission is complete, the clone "disperses": it returns a structured report back to the main Controller, who uses that knowledge for planning, implementation, review, or acceptance.

The point is not to use more agents.

The point is to keep each agent's context clean and each agent's mission narrow.

## Core Principles

### 1. The Controller Stays In Charge

The inline agent is the Controller. It holds the user goal, task order, accepted decisions, allowed write scopes, accumulated discoveries, and final judgment.

Clones provide focused intelligence or scoped implementation. The Controller decides what to accept, reject, retry, or hand off.

### 2. Clean-Context Isolation

A clone starts with a blank slate. It does not carry the baggage of the current chat session, previous false starts, tool noise, or the implementer's assumptions.

It only knows its mission, scope, constraints, relevant context, and expected output.

### 3. Evidence-Grounded Intelligence

A clone cannot guess or say "looks good."

It must return evidence: exact file paths, code references, schemas, diffs, command output, tests, artifacts, and explicit uncertainty.

Its most important job is to find the rules that must not be broken: interfaces, invariants, downstream consumers, time boundaries, schema stability, identity joins, train/serve parity, and other hidden contracts.

### 4. Many Readers, One Writer

Read-only clones can run in parallel when they inspect independent, stable parts of the system.

Coding clones run sequentially. One coding clone writes one bounded task at a time. Testing and review happen only after the work is stable.

## Basic Workflow

```text
1. Controller decides whether clones would help.
2. Read-only clones gather evidence if context is unclear.
3. Controller turns findings into bounded tasks.
4. One coding clone implements one task.
5. Read-only clones test, verify, or review the stable result.
6. Controller accepts, rejects, or creates a bounded fix task.
7. Useful discoveries carry forward.
```

## When To Use

Use `shadow-clone` when work is large, risky, cross-cutting, or context-heavy.

Good signals:

- multiple files or components
- unknown downstream consumers
- schemas, APIs, CLIs, artifacts, model features, or data contracts
- smoke tests, canaries, snapshots, generated outputs, or artifact checks
- unclear invariants or hidden coupling
- a need for fresh-context review

Do not use it for tiny, obvious, mechanical changes where coordination costs more than it helps.

## Rule Of Thumb

Numbers are warning lights, not decisions.

Consider `shadow-clone` when a task involves:

- 3+ files likely touched
- 4+ files needing research
- 300+ lines of existing code to inspect
- 2+ components
- unknown or multiple downstream consumers
- smoke, canary, snapshot, artifact, or generated-output verification
- schema, API, CLI, model, data, or migration risk

A one-file change can still need `shadow-clone` if it touches a risky interface. A many-file change may not need it if it is mechanical and low-risk.
