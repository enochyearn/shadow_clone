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

Read-only research clones can run in parallel when they inspect independent, stable parts of the system. Review clones run in parallel only when the controller chooses that for a stable diff.

Coding clones run sequentially, even when they touch separate files. One coding clone writes one bounded task at a time. Testing and review happen only after the work is stable.

## Basic Workflow

```text
1. Controller decides whether clones would help.
2. Read-only clones gather evidence if context is unclear.
3. Controller turns findings into bounded tasks.
4. One coding clone implements one task.
5. Read-only clones test, verify, or review the stable result.
6. Controller accepts, rejects, or creates a bounded fix task.
7. Useful discoveries carry forward.
8. After the last task, the controller verifies the combined change and reports what is and is not proven.
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

The full table of signals and thresholds is in `SKILL.md`.

## Install

Clone this repository into your agent's skills folder, in a folder named `shadow-clone`, with a hyphen. The Agent Skills format expects a skill's folder name to match its `name`, and this repository's name uses an underscore.

### Codex

Recent Codex versions load user skills from `~/.agents/skills` and repository skills from `.agents/skills`. Skill folders have changed between versions, so check the [Codex skills documentation](https://developers.openai.com/codex/skills) for yours.

For all your projects:

```bash
git clone https://github.com/enochyearn/shadow_clone.git ~/.agents/skills/shadow-clone
```

For one project, from its root:

```bash
git clone https://github.com/enochyearn/shadow_clone.git .agents/skills/shadow-clone
```

Restart Codex if the skill does not appear.

### Other agents

Any agent that supports the Agent Skills format can load it. Put the `shadow-clone` folder in that agent's skills directory.

## Use

- **Codex:** type `$shadow-clone` followed by your task. The [Codex subagents documentation](https://developers.openai.com/codex/subagents) says Codex starts subagents only when asked, so say that subagents are allowed. The skill's default prompt in the Codex app already does.
- **Other agents:** invoke the skill by name, or let the agent choose it from its description.

The skill works best on a host that can start subagents. Without them, it runs the same phases inline, one at a time.

## Files

| File | What it holds |
|---|---|
| `SKILL.md` | The instructions the agent loads |
| `references/templates.md` | The Clone Rules block, dispatch prompt, task docket, and report templates |
| `evals/scenarios.md` | Scenarios for testing the skill |
| `agents/openai.yaml` | Display name and default prompt for the Codex app |

## Companion Skills

`SKILL.md` can hand work to `planning-and-task-breakdown`, `writing-plans`, `subagent-driven-development`, and `recursive-decomposition` when they are installed. They are separate skills and are not included here. Without them, the skill uses its own templates.

## Testing

`evals/scenarios.md` lists scenarios with the expected behavior and the signs of failure. To test a change, install the skill in a fresh session, give the agent a scenario's input, and compare what it does with the expected behavior.
