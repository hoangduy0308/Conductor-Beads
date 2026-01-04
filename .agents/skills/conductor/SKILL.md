---
name: conductor
description: Coordinates spec-first, context-driven development by managing tracks, specs, and TDD-focused implementation for projects with a conductor/ directory when users want organized, status-aware workflows and documentation-code alignment.
---

# Conductor: Context-Driven Development

Measure twice, code once.

## Overview

| Aspect | Description |
|--------|-------------|
| **Input** | Project with `conductor/` directory, user intent for features/bugs |
| **Output** | Tracks with specs, phased plans, TDD implementation, status tracking |
| **Done when** | Track completed, tests passing, documentation in sync |

Conductor enables context-driven development by:
1. Establishing project context (product vision, tech stack, workflow)
2. Organizing work into "tracks" (features, bugs, improvements)
3. Creating specs and phased implementation plans
4. Executing with TDD practices and progress tracking
5. Multi-agent parallel execution via orchestrating-beads skill

**Integrations**: Interoperable with Gemini CLI extension and Claude Code. Integrates with Beads for persistent task memory.

---

## Conventions

| Convention | Description |
|------------|-------------|
| **Track ID** | Format: `<shortname>_YYYYMMDD` (e.g., `auth_20241225`) |
| **Directory** | `conductor/tracks/<track_id>/` with spec.md, plan.md, metadata.json |
| **Status markers** | `[ ]` pending, `[~]` in-progress, `[x]` done, `[!]` blocked, `[-]` skipped |
| **Parallel annotation** | `<!-- execution: parallel -->` in plan.md for concurrent tasks |
| **File annotation** | `<!-- files: path/to/file.ts -->` for task scope |
| **Commands** | `/conductor-*` prefix for all commands |
| **Commits** | Local only; never auto-push (users decide when to push) |

---

## Quickstart

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `/conductor-setup` | Initialize product.md, tech-stack.md, workflow.md, tracks.md |
| 2 | `/conductor-newtrack` | Create spec + plan for a feature/bug |
| 3 | `/conductor-implement` | Execute tasks with TDD + progress tracking |
| 4 | `/conductor-status` | Check progress overview |
| 5 | `/conductor-refresh` | Sync docs with codebase changes |

---

## When to Use Which Entry Point

| Use Case | Entry Point |
|----------|-------------|
| Bug fix (1-5 files, known location) | `/conductor-newtrack` |
| Small chore/refactor | `/conductor-newtrack` |
| Large feature (multi-module) | `planning` skill |
| External integration (APIs, payments) | `planning` skill |
| Unknown scope (needs discovery) | `planning` skill |

See [references/workflows.md](references/workflows.md) for the full workflow diagram.

---

## Core Workflows

### /conductor-setup

| Aspect | Description |
|--------|-------------|
| **Input** | Project directory (greenfield or brownfield) |
| **Output** | `conductor/` with product.md, tech-stack.md, workflow.md, tracks.md |
| **Done when** | setup_state.json has `"last_successful_step": "complete"` |

1. Detect project type (brownfield/greenfield)
2. **Deep Research Phase**: 
   - Greenfield: `exa-code` + Oracle → tech recommendations
   - Brownfield: `gkg repo_map` + Oracle → code analysis
3. Interactive Q&A for context files
4. Create initial track if approved
5. Commit: `conductor(setup): Initialize conductor`

### /conductor-newtrack

| Aspect | Description |
|--------|-------------|
| **Input** | Feature/bug description |
| **Output** | Track directory with spec.md, plan.md, metadata.json |
| **Done when** | Track visible in `conductor/tracks.md` with status `new` |

1. Verify setup complete
2. Interactive spec generation (3-5 questions)
3. Generate phased plan with TDD tasks
4. Create track artifacts
5. **If Beads enabled**: Create epic and tasks
6. Commit: `conductor(track): Add <track_id>`

### /conductor-implement

| Aspect | Description |
|--------|-------------|
| **Input** | Track ID (or auto-detect in-progress track) |
| **Output** | Completed tasks, passing tests, commits |
| **Done when** | All plan.md tasks marked `[x]`, phase verifications pass |

1. Load track context (spec.md, plan.md)
2. Resume from implement_state.json if exists
3. **If bead-backed**: Delegate to `orchestrating-beads` skill
4. **Else**: Execute tasks sequentially with TDD
5. Update markers and commit after each task

### /conductor-refresh

| Aspect | Description |
|--------|-------------|
| **Input** | Scope (all/track_id/specific files) |
| **Output** | Updated context docs reflecting current codebase |
| **Done when** | Docs synced, refresh_state.json updated |

1. Analyze codebase changes since last refresh
2. Update product.md, tech-stack.md if affected
3. Update track specs/plans if implementation diverged
4. Commit: `conductor(refresh): Sync context`

---

## Context Loading

When this skill activates, load these files:
1. `conductor/product.md` - Product vision and goals
2. `conductor/tech-stack.md` - Technology constraints
3. `conductor/workflow.md` - Development methodology (TDD, commits)
4. `conductor/tracks.md` - Current work status

For active tracks, also load:
- `conductor/tracks/<track_id>/spec.md`
- `conductor/tracks/<track_id>/plan.md`

---

## Beads Integration

Conductor **always checks for Beads** and offers integration when available.

### Detection Rule

Only use `bd` commands if ALL conditions are met:
1. `which bd` returns a valid path
2. `conductor/beads.json` exists with `"enabled": true`

### If Beads is NOT available
- DO NOT run any `bd` commands
- Use only plan.md markers for task tracking
- All commands work normally without Beads

### If Beads IS available
- Tracks become Beads epics
- Tasks sync to Beads for persistent memory
- Use `bd ready` for task selection
- Notes survive context compaction

### Bead-Backed Tracks

When `metadata.json` has `beads_epic`:
- Task tracking is in Beads (not plan.md checkboxes)
- `/conductor-implement` delegates to `orchestrating-beads` skill
- Status derived from `bd list`, `bv --robot-triage`

See [references/beads-integration.md](references/beads-integration.md) for CLI examples.

---

## Proactive Behaviors

| Trigger | Action |
|---------|--------|
| New session | Check for in-progress tracks, offer to resume |
| Task completion | Suggest next task or phase verification |
| Blocked detection | Alert user and suggest alternatives |
| All tasks complete | Congratulate and offer archive/cleanup |
| Stale context (>2 days) | Suggest `/conductor-refresh` |
| Beads available | Offer integration during setup |
| Complex feature | Suggest `planning` skill instead |

---

## Intent Mapping

| User Intent | Command/Skill |
|-------------|---------------|
| "Set up this project" | `/conductor-setup` |
| "Create a new feature" (simple) | `/conductor-newtrack [desc]` |
| "Plan a big feature" | `planning` skill |
| "Start working" / "Implement" | `/conductor-implement [id]` |
| "What's the status?" | `/conductor-status` |
| "Undo that" / "Revert" | `/conductor-revert` |
| "Check for issues" | `/conductor-validate` |
| "This is blocked" | `/conductor-block` |
| "Skip this task" | `/conductor-skip` |
| "This needs revision" | `/conductor-revise` |
| "Save context" / "Handoff" | `/conductor-handoff` |
| "Archive completed" | `/conductor-archive` |
| "Export summary" | `/conductor-export` |
| "Docs are outdated" | `/conductor-refresh` |

---

## Related Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| **beads** | Foundation for bd CLI | All bead-backed work |
| **planning** | Discovery, risk assessment | Complex features |
| **filing-beads** | Create structured beads | After planning |
| **reviewing-beads** | Quality gate for beads | Before execution |
| **orchestrating-beads** | Multi-agent execution | Bead-backed tracks |

---

## References

- **Detailed workflows**: [references/workflows.md](references/workflows.md) - Step-by-step command execution
- **Directory structure**: [references/structure.md](references/structure.md) - File layout and status markers
- **Beads integration**: [references/beads-integration.md](references/beads-integration.md) - CLI examples and configuration
