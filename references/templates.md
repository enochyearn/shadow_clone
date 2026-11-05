# Shadow Clone Templates

Copy the Clone Rules block and the matching template into each clone prompt. Clones start with a fresh context and never see `SKILL.md`.

## Contents

1. Clone Rules
2. Clone Dispatch Prompt
3. Task Docket for Coding Clone
4. Shadow Research Brief
5. Task Routing Table
6. Coding Clone Report
7. Testing / Verification Report
8. Review Report
9. Debug Diagnosis Report
10. Accumulated Discoveries Log

Which report each clone returns:

| Clone | Returns |
|---|---|
| Read-only, research mode | Shadow Research Brief |
| Read-only, testing / verification mode | Testing / Verification Report |
| Read-only, review mode | Review Report |
| Read-only, debug diagnosis mode | Debug Diagnosis Report |
| Coding clone, any write scope | Coding Clone Report |

## 1. Clone Rules

Paste this block into every clone prompt, unchanged.

```markdown
# Clone Rules
- Stay inside the scope in this prompt. If you need to leave it or make a design decision, stop and report the question instead.
- Edit only inside your allowed write scope. Read-only clones edit nothing.
- Do not start other agents. Do not commit, push, or switch branches unless this prompt says so.
- Search before reading broadly.
- Back every finding with evidence (file path and line, command output, test name, or diff) and mark uncertainty.
- Use the expected output format. Omit sections that do not apply, aim for about one screen, and put bulky evidence in a file and give its path.
- Stop when the stop condition is met, and report.
```

## 2. Clone Dispatch Prompt

Use this structure for every clone. For a coding clone, the Task Docket supplies the Mission, Context, Scope, and Constraints.

```markdown
# Your Role
[Coding Clone / Read-Only Clone]

# Mode or Write Scope
[Read-only mode: research / testing-verification / review / debug-diagnosis]
[Coding write scope: implementation / test-only / fix]

# Mission
[Exact bounded task]

# Context
[Only necessary context]

# Scope
In scope:
- `path`

Out of scope:
- `path` or behavior

# Constraints
[Read-only or allowed write scope, commands allowed/disallowed, must-preserve rules]

# Clone Rules
[Paste the Clone Rules block from section 1, unchanged]

# Expected Output
[Template or exact format]

# Stop Condition
[When to stop and report back]
```

## 3. Task Docket for Coding Clone

```markdown
# Task Docket: [Task Name]

## Goal
[What this task must accomplish.]

## Non-Goals
[What this task must not do.]

## Relevant Context
[Only necessary excerpts from research briefs, user goals, accepted decisions, and discoveries.]

## Allowed Write Scope
- `path`

## Read Scope
- `path`

## Must-Preserve Interfaces / Invariants
- [interface/invariant]

## Acceptance Criteria
- [criterion]

## Verification Commands
- `command`

## Stop Condition
Stop after this task. Report with the Coding Clone Report. If you need to leave scope or make a design decision, stop and report the question instead.
```

## 4. Shadow Research Brief

```markdown
# Shadow Research Brief: [Component / Surface]

## Mission

**Focus:** [component / producer / consumer / integration / verification]

**Controller question:** [question]

**Scope boundary:** [in-scope paths/components]

**Out of scope:** [not inspected]

## Executive Summary

- [3-6 key findings]

## Evidence Inspected

| Path | Why It Matters |
|---|---|
| `path` |  |

## Entrypoints

| Entrypoint | Type | Location | Notes |
|---|---|---|---|
|  | function / class / CLI / job / API / script / notebook / test | `path` |  |

## Inputs

| Input | Source | Shape / Required Fields | Assumptions | Evidence |
|---|---|---|---|---|
|  |  |  |  | `path` |

## Outputs

| Output | Destination / Interface | Shape / Schema / Layout | Consumers | Evidence |
|---|---|---|---|---|
|  |  |  |  | `path` |

## Consumers and Dependencies

### Upstream Dependencies

- [dependency] - evidence: `path`

### Downstream Consumers

- [consumer] - evidence: `path`

## Interface / Invariant Notes

- [specific thing future implementers/reviewers must preserve] - evidence: `path` - confidence: High/Medium/Low

## Verification Surface

| Command / Test / Check | What It Proves | What It Does Not Prove |
|---|---|---|
| `command or test path` |  |  |

## Risks and Unknowns

- [risk/unknown]

## Planning Implications

- [task split recommendation]
- [review lens recommendation]
- [verification recommendation]

## Confidence

**Overall confidence:** High / Medium / Low

**Why:** [brief explanation]
```

## 5. Task Routing Table

```markdown
# Shadow Clone Routing Table

| Task | Size | Depends On | Write Scope | Review Lenses | Verification | Next Skill / Next Step |
|---|---:|---|---|---|---|---|
| T1 | XS/S/M/L/XL | none/T# | `path` | spec + quality | `pytest ...` | subagent-driven-development, if installed |
```

Sizes follow the scale in `SKILL.md` (Scale Process to Risk). Split L and XL tasks before implementation. Tasks run one at a time, in dependency order.

## 6. Coding Clone Report

```markdown
# Coding Clone Report: [Task]

## Changed Files
- `path` - [what changed]

## Commands Run

| Command | Result | Evidence |
|---|---|---|
| `command` | PASS/FAIL | output summary |

## Acceptance Criteria

| Criterion | Met? | Evidence |
|---|---|---|
| [criterion] | Yes/No/Partly | `path` / output |

## Uncertainties and Risks
- [item]

## Questions for Controller
- [scope or design question that stopped the work, if any]

## New Discoveries
- [discovery]
```

## 7. Testing / Verification Report

```markdown
# Testing / Verification Report: [Task]

## Commands / Checks Run

| Command / Check | Result | Evidence |
|---|---|---|
| `command` | PASS/FAIL/SKIPPED | output/artifact path |

## What Is Proven
- [claim backed by evidence]

## What Is Not Proven
- [gap]

## Coverage Gaps / Edge Cases
- [gap]

## Side Effects Observed
- [caches, snapshots, generated files, or external state written; or none]

## Verdict
PASS / FAIL / INCONCLUSIVE
```

## 8. Review Report

```markdown
# Shadow Review Report: [Task]

## Review Lenses Applied
- spec compliance
- interface / invariant
- code quality
- test adequacy
- verification

## Findings

| Severity | Lens | Finding | Evidence | Required Fix? |
|---|---|---|---|---|
| Critical/Important/Minor/Nit |  |  | `path` / output | Yes/No |

## Verdict
ACCEPT / ACCEPT WITH MINOR FIXES / REJECT

## Notes for Controller
- [triage notes]
```

## 9. Debug Diagnosis Report

```markdown
# Debug Diagnosis Report: [Failure]

## Symptom
[Exact error, failing test, or observed versus expected behavior]

## Reproduction
`command` - [result]

## Evidence
- [finding] - evidence: `path:line` / output

## Root Cause
[cause] - confidence: High/Medium/Low

## Smallest Fix Path
- [change] in `path`

## What Would Disprove This
- [check]
```

## 10. Accumulated Discoveries Log

Keep this log in a scratch file outside the repository, or in the plan file. Never commit it.

```markdown
# Accumulated Discoveries

## Codebase Conventions
- [convention] - discovered in [task/clone]

## Gotchas
- [gotcha] - discovered in [task/clone]

## Failed Assumptions
- [assumption] -> [actual truth] - discovered in [task/clone]

## Decisions Made
- [decision and rationale] - made by controller after [task/clone]

## Verification Commands That Matter
- `command` - proves [what] - discovered in [task/clone]

## Cross-Cutting Concerns
- [concern affecting multiple tasks]
```
