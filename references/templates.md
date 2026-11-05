# Shadow Clone Templates

Copy the matching template into each clone prompt. Clones start with a fresh context and never see `SKILL.md`.

## Contents

- Output Templates
  1. Shadow Research Brief
  2. Task Routing Table
  3. Task Docket for Coding Clone
  4. Testing / Verification Report
  5. Review Report
  6. Accumulated Discoveries Log
- Clone Dispatch Prompt Template

## Output Templates

### 1. Shadow Research Brief

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

### 2. Task Routing Table

```markdown
# Shadow Clone Routing Table

| Task | Size | Depends On | Write Scope | Sequential Reason | Review Lenses | Verification | Next Skill / Next Step |
|---|---:|---|---|---|---|---|---|
| T1 | S/M/L | none/T# | `path` | shared file / current diff / command output | spec + quality | `pytest ...` | subagent-driven-development |
```

Sizing guide:

```text
XS: 1 file, obvious local change
S: 1-2 files, one concern
M: 3-5 files, one feature slice
L: 5-8 files, split if possible
XL: 8+ files or multiple independent subsystems; must split before implementation
```

### 3. Task Docket for Coding Clone

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
Stop after this task. Report changed files, commands run, output summary, risks, and new discoveries.
```

### 4. Testing / Verification Report

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

## Verdict
PASS / FAIL / INCONCLUSIVE
```

### 5. Review Report

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

### 6. Accumulated Discoveries Log

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

## Clone Dispatch Prompt Template

Use this structure for every clone.

```markdown
# Your Role
[Coding Clone / Read-Only Clone]

# Mode
[research / testing-verification / review / debug-diagnosis / implementation]

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

# Expected Output
[Template or exact format]

# Stop Condition
[When to stop and report back]
```
