# AGENTS.md

This file is the entry point for the `conductor` skill and related Beads skills. It helps AI agents understand how to use Conductor for context-driven development.

## What is Conductor?

Conductor is a **spec-first development methodology** that organizes work into **tracks** (scoped units of work like features, bugs, or refactors, each with a `spec.md` and `plan.md`). It integrates with Beads for persistent task memory across sessions.

## Skills Overview

| Skill | Purpose |
|-------|---------|
| `conductor` | Entry point for spec-first tracks and `/conductor-*` workflows |
| `beads` | Foundation skill for `bd` CLI and persistent task graph |
| `planning` | Discovery, risk assessment, spike execution for complex features |
| `filing-beads` | Convert plans into epics/issues with dependencies |
| `reviewing-beads` | Quality gate for clarity and completeness |
| `orchestrating-beads` | Multi-agent parallel execution of bead graphs |
| `skill-creator` | Create new domain-specific skills when needed |

### When to Use Which

- **Day-to-day work**: Use `conductor` skill (setup, new tracks, implementation, status)
- **Complex features**: Use `planning` skill for discovery and risk assessment
- **Multi-agent execution**: Use full pipeline: `planning → filing-beads → reviewing-beads → orchestrating-beads`
- **Fine-grained task ops**: Use `beads` skill for direct `bd`/`bv` operations
- **Extending ecosystem**: Use `skill-creator` for recurring patterns

## Quick Start

1. **Setup**: Run `/conductor-setup` (Claude) or `/conductor:setup` (Gemini) to initialize
   - **Deep Research**: Setup now uses `exa-code` + `Oracle` for Greenfield, `gkg` + `Oracle` for Brownfield
2. **Create Track**: Run `/conductor-newtrack` to create a new feature/bug track
3. **Implement**: Run `/conductor-implement` to execute tasks from the plan
4. **Status**: Run `/conductor-status` to check progress

## Setup Deep Research

`/conductor-setup` now performs deep analysis before Q&A:

| Project Type | Tools Used | Purpose |
|--------------|------------|---------|
| **Greenfield** | `exa-code` + `Oracle` | Research tech stacks, best practices, architecture patterns |
| **Brownfield** | `gkg repo_map` + `Oracle` | Analyze existing code structure, conventions, dependencies |

This ensures Q&A suggestions are **informed by research**, not generic options.

## Key Files to Read

When Conductor is set up, always read these files first:
- `conductor/product.md` - Product vision and goals
- `conductor/tech-stack.md` - Technology constraints
- `conductor/workflow.md` - TDD methodology and commit rules
- `conductor/tracks.md` - Current work status

For active tracks:
- `conductor/tracks/<track_id>/spec.md` - What to build
- `conductor/tracks/<track_id>/plan.md` - How to build it (phased tasks)

## Beads Integration

Conductor **checks for Beads** and uses it when available. The `beads` skill and Beads pipeline skills wrap `bd`/`bv` CLI; prefer skills for automation.

### Detection (before using `bd`/`bv` CLI)

Only run `bd`/`bv` commands if ALL conditions are true:
1. `which bd` returns valid path
2. `conductor/beads.json` exists with `"enabled": true`

### If Beads is Available

- Use `bd ready` to find tasks with no blockers
- Use `bd update <id> --notes "..."` to preserve context across sessions
- Use `bd close <id>` when task is complete

### If Beads is NOT Available

- Ignore bead-backed sections
- Use `plan.md` checkboxes for task status
- `/conductor-implement` works normally without Beads

## Beads-Backed Pipeline

For complex features, use the full planning pipeline instead of `/conductor-newtrack`:

```
planning → filing-beads → reviewing-beads → orchestrating-beads
```

| Skill | Purpose |
|-------|---------|
| `planning` | Discovery, risk assessment, spike execution |
| `filing-beads` | Convert plans into epics/issues with dependencies |
| `reviewing-beads` | Quality gate for clarity and completeness |
| `orchestrating-beads` | Multi-agent parallel execution |

> **Note**: These are skills, not shell commands. Invoke them via the agent skill interface.

**Bead-backed tracks**: When a track has `beads_epic` in its `metadata.json`:
- Task status lives in Beads (not plan.md checkboxes)
- `/conductor-implement` delegates to `orchestrating-beads` skill
- Use `bd ready` and `bv --robot-*` for task/graph navigation

## Task Workflow

1. Mark task `[~]` in progress in plan.md
2. Follow TDD: Write tests → Implement → Refactor
3. Ensure >80% coverage (or as specified in workflow.md)
4. Commit with conventional message: `type(scope): description`
5. Mark task `[x]` complete
6. **NEVER run `git push`** - all commits stay local

## Status Markers

- `[ ]` - Pending
- `[~]` - In Progress  
- `[x]` - Completed
- `[!]` - Blocked

## Available Commands

| Command | Purpose |
|---------|---------|
| `/conductor-setup` | Initialize project with Conductor |
| `/conductor-newtrack` | Create new feature/bug track |
| `/conductor-implement` | Execute tasks from plan |
| `/conductor-status` | Check project/track status |
| `/conductor-validate` | Validate conductor setup |
| `/conductor-block` | Mark task as blocked |
| `/conductor-skip` | Skip a task |
| `/conductor-revise` | Revise spec or plan |
| `/conductor-revert` | Undo recent changes |
| `/conductor-archive` | Archive completed track |
| `/conductor-export` | Export track summary |
| `/conductor-refresh` | Sync context with codebase |
| `/conductor-handoff` | Save context for session handoff |

## Proactive Behaviors

- On new session: Check for in-progress tracks, offer to resume
- On task completion: Suggest next task or phase verification
- On blocked detection: Alert user and suggest alternatives
- On complex feature request: Suggest `planning` skill instead of classic newtrack
