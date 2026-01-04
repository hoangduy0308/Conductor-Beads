---
name: planning
description: Generates beads-backed feature roadmaps via a discovery → synthesis → verification → decomposition pipeline for complex, multi-module features requiring execution-ready plans integrated with Conductor tracks and beads/bv.
---

# Beads-based Feature Planning

Systematic discovery → synthesis → verification → decomposition for feature planning.

**Prerequisite**: `beads` skill / `bd` CLI initialized in this repo (see [beads skill](../beads/SKILL.md))

**When to use**: Complex features requiring discovery, risk assessment, or multi-agent execution.
**Alternative**: Use `/conductor-newtrack` directly for simple bugs/chores.

**Feature naming**: Use kebab-case slugs (e.g., `user-auth`, `payment-integration`)

## Pipeline Overview

```
USER REQUEST → Discovery → Synthesis → Verification → Decomposition → Refinement → Validation → Track Planning → Ready Plan
```

| Phase             | Tool                                     | Output                              |
| ----------------- | ---------------------------------------- | ----------------------------------- |
| 1. Discovery      | Parallel sub-agents, gkg, Librarian, exa | Discovery Report                    |
| 2. Synthesis      | Oracle                                   | Approach + Risk Map                 |
| 3. Verification   | Spikes via orchestrating-beads skill     | Validated Approach + Learnings      |
| 4. Decomposition  | filing-beads skill                       | Beads graph in `.beads/issues.jsonl` |
| 5. Refinement     | reviewing-beads skill                    | Polished beads ready for workers    |
| 6. Validation     | bv + Oracle                              | Validated dependency graph          |
| 7. Track Planning | bv --robot-plan                          | Execution plan with parallel tracks |

## Prerequisites

Before using this skill, ensure:
1. `/conductor-setup` has been run (conductor/ directory exists)
2. Read `conductor/product.md`, `conductor/tech-stack.md` for project context

## Conventions

- **`Task()`**: Create a sub-agent to execute work in parallel. Each Task() runs independently.
- **`oracle()`**: Use the Oracle tool for analysis/synthesis. Expects `task`, `context`, and `files` parameters.
- **`skill("name")`**: Load another skill for specialized workflows.
- **Feature slug**: Use kebab-case (e.g., `user-auth`, `payment-integration`)

## Quickstart

1. **Phase 1 – Discovery** → save `history/<feature>/discovery.md`
2. **Phase 2 – Synthesis** (Oracle) → `history/<feature>/approach.md`
3. **Phase 3 – Verification** → create spikes for HIGH risk items via `orchestrating-beads`
4. **Phase 4 – Decomposition** → use `filing-beads` to create beads
5. **Phase 5 – Refinement** → run `reviewing-beads` to polish
6. **Phase 6 – Validation** → run `bv` to validate graph
7. **Phase 7 – Track Planning** → generate `execution-plan.md`
8. **Phase 8 – Create Track** → `/conductor-newtrack --import`

## Phase 1: Discovery (Parallel Exploration)

**Input**: User feature request, `conductor/product.md`, `conductor/tech-stack.md`
**Output**: `history/<feature>/discovery.md`
**Done when**: Architecture snapshot, existing patterns, constraints, and external references captured

Launch parallel sub-agents to gather codebase intelligence:

```
Task() → Agent A: Architecture snapshot (gkg repo_map)
Task() → Agent B: Pattern search (find similar existing code)
Task() → Agent C: Constraints (package.json, tsconfig, deps)
Librarian → External patterns ("how do similar projects do this?")
exa → Library docs (if external integration needed)
```

### Discovery Report

Save to `history/<feature>/discovery.md`. See `references/discovery-template.md` for full template.

## Phase 2: Synthesis (Oracle)

**Input**: `history/<feature>/discovery.md`
**Output**: `history/<feature>/approach.md`
**Done when**: Gap Analysis, Approach Options, and Risk Map completed using template

Feed Discovery Report to Oracle for gap analysis:

```
oracle(
  task: "Analyze gap between current codebase and feature requirements",
  context: "Discovery report attached. User wants: <feature>",
  files: ["history/<feature>/discovery.md"]
)
```

Oracle produces:

1. **Gap Analysis** - What exists vs what's needed
2. **Approach Options** - 1-3 strategies with tradeoffs
3. **Risk Assessment** - LOW / MEDIUM / HIGH per component

### Risk Classification

| Level  | Criteria                      | Verification                 |
| ------ | ----------------------------- | ---------------------------- |
| LOW    | Pattern exists in codebase    | Proceed                      |
| MEDIUM | Variation of existing pattern | Interface sketch, type-check |
| HIGH   | Novel or external integration | Spike required               |

### Risk Indicators

```
Pattern exists in codebase? ─── YES → LOW base
                            └── NO  → MEDIUM+ base

External dependency? ─── YES → HIGH
                     └── NO  → Check blast radius

Blast radius >5 files? ─── YES → HIGH
                       └── NO  → MEDIUM
```

Save to `history/<feature>/approach.md`. See `references/approach-template.md` for full template.

## Phase 3: Verification (Risk-Based)

**Input**: `history/<feature>/approach.md` with Risk Map
**Output**: `.spikes/<feature>/` with validated code, updated `approach.md`
**Done when**: All HIGH risk items have spikes completed with YES/NO answers

### For HIGH Risk Items → Create Spike Beads

Spikes are mini-plans executed via the orchestrating-beads skill:

```bash
bd create "Spike: <question to answer>" -t epic -p 0
bd create "Spike: Test X" -t task --deps <spike-epic>
bd create "Spike: Verify Y" -t task --deps <spike-epic>
```

### Spike Bead Template

See `references/spike-template.md` for the full spike bead template.

### Execute Spikes

Use the orchestrating-beads skill (Spike Mode):

1. `bv --robot-plan` to parallelize spikes
2. `Task()` per spike with time-box
3. Workers write to `.spikes/<feature>/<spike-id>/`
4. Close with learnings: `bd close <id> --reason "<result>"`

### Aggregate Spike Results

```
oracle(
  task: "Synthesize spike results and update approach",
  context: "Spikes completed. Results: ...",
  files: ["history/<feature>/approach.md"]
)
```

Update approach.md with validated learnings.

## Phase 4: Decomposition (filing-beads skill)

**Input**: Validated `approach.md`, spike learnings from `.spikes/`
**Output**: Beads in `.beads/issues.jsonl`
**Done when**: All work items created as beads with acceptance criteria and file scopes

Load the filing-beads skill and create beads with embedded learnings:

```bash
skill("filing-beads")
```

See [filing-beads skill](../filing-beads/SKILL.md) for detailed bead creation guidelines.

Each bead MUST include spike learnings (if applicable), reference to `.spikes/` code for HIGH risk items, clear acceptance criteria, and file scope for track assignment. See `references/bead-with-learnings-example.md` for a full example.

## Phase 5: Refinement (reviewing-beads skill)

**Input**: Beads in `.beads/issues.jsonl`
**Output**: Polished beads with clear acceptance criteria
**Done when**: reviewing-beads reports no critical issues (clarity, scope, dependencies checked)

Run `skill("reviewing-beads")` to polish beads. See [reviewing-beads skill](../reviewing-beads/SKILL.md) for the full quality checklist.

## Phase 6: Validation

**Input**: Polished beads from Phase 5
**Output**: Validated dependency graph with no cycles or orphans
**Done when**: `bv --robot-insights` shows no cycles, `bv --robot-plan` has no unassigned beads

Run bv analysis and fix issues. If issues found, re-run `reviewing-beads` skill.

```bash
bv --robot-suggest   # Find missing dependencies
bv --robot-insights  # Detect cycles, bottlenecks
bv --robot-priority  # Validate priorities
bd dep add <from> <to>      # Add missing deps
bd dep remove <from> <to>   # Break cycles
```

Oracle final review:

```
oracle(
  task: "Review plan completeness and clarity",
  files: [".beads/"]
)
```

## Phase 7: Track Planning

**Input**: Validated bead graph, `bv --robot-plan` output
**Output**: `history/<feature>/execution-plan.md`
**Done when**: All tracks have non-overlapping file scopes, no unassigned beads

Creates an **execution-ready plan** for the orchestrating-beads skill. The `execution-plan.md` is the canonical input for orchestrating-beads.

1. Get tracks: `bv --robot-plan 2>/dev/null | jq '.plan.tracks'`
2. Assign non-overlapping file scopes per track
3. Generate agent names (adjective+noun: BlueLake, GreenCastle, etc.)
4. Save to `history/<feature>/execution-plan.md`. See `references/execution-plan-template.md`.

Final check:

```bash
bv --robot-insights 2>/dev/null | jq '.Cycles'  # Must be empty
bv --robot-plan 2>/dev/null | jq '.plan.unassigned'  # Must be empty
```

## Phase 8: Create Conductor Track

**Input**: `execution-plan.md`, beads in `.beads/`
**Output**: Conductor track in `conductor/tracks/<track_id>/`
**Done when**: Track registered in `conductor/tracks.md` with `beads_epic` in metadata

After planning is complete, create the Conductor track to register it:

```bash
/conductor-newtrack --import "history/<feature>/execution-plan.md"
```

This creates:
- `conductor/tracks/<track_id>/spec.md` (high-level, references approach.md)
- `conductor/tracks/<track_id>/plan.md` (meta-level, references beads)
- `conductor/tracks/<track_id>/metadata.json` (with `beads_epic`)
- Entry in `conductor/tracks.md`

## Output Artifacts

| Artifact          | Location                              | Purpose                            |
| ----------------- | ------------------------------------- | ---------------------------------- |
| Discovery Report  | `history/<feature>/discovery.md`      | Codebase snapshot                  |
| Approach Document | `history/<feature>/approach.md`       | Strategy + risks                   |
| Spike Code        | `.spikes/<feature>/`                  | Reference implementations          |
| Spike Learnings   | Embedded in beads                     | Context for workers                |
| Beads             | `.beads/issues.jsonl`                 | Executable work items              |
| Execution Plan    | `history/<feature>/execution-plan.md` | Track assignments for orchestrator |

## Quick Reference

### Tool Selection

| Need               | Tool                                    |
| ------------------ | --------------------------------------- |
| Codebase structure | `mcp__gkg__repo_map`                    |
| Find definitions   | `mcp__gkg__search_codebase_definitions` |
| Find usages        | `mcp__gkg__get_references`              |
| External patterns  | `librarian`                             |
| Library docs       | `mcp__exa__get_code_context_exa`        |
| Gap analysis       | `oracle`                                |
| Create beads       | `skill("filing-beads")` + `bd create`   |
| Validate graph     | `bv --robot-*`                          |

### Common Mistakes

- **Skipping discovery** → Plan misses existing patterns
- **No risk assessment** → Surprises during execution
- **No spikes for HIGH risk** → Blocked workers
- **Missing learnings in beads** → Workers re-discover same issues
- **No bv validation** → Broken dependency graph
