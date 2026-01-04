---
name: beads
description: Tracks complex multi-session work as a git-backed dependency graph using the bd CLI when tasks span multiple sessions, have non-trivial dependencies, or need persistent context that survives conversation compaction.
---

# Beads - Persistent Task Memory for AI Agents

Graph-based issue tracker that survives conversation compaction. Unlike TodoWrite (session-scoped), beads persists across compactions and tracks complex dependencies.

## Overview

| Aspect | Description |
|--------|-------------|
| **Input** | Work items needing tracking across sessions, dependency relationships |
| **Output** | Git-backed task graph in `.beads/issues.jsonl` with full audit trail |
| **Done when** | Tasks tracked, dependencies mapped, ready work identified |

**Core Capabilities**:
- 📊 **Dependency Graphs**: Track what blocks what (blocks, parent-child, related)
- 💾 **Compaction Survival**: Tasks persist when conversation history is compacted
- 🐙 **Git Integration**: Issues versioned in `.beads/issues.jsonl`
- 🔍 **Smart Discovery**: Auto-finds ready work (`bd ready`), blocked work (`bd blocked`)
- 📝 **Audit Trails**: Complete history of status changes, notes, and decisions

**When to Use bd vs TodoWrite**:

| Question | Answer | Use |
|----------|--------|-----|
| Will I need this context in 2 weeks? | YES | bd |
| Could conversation history get compacted? | YES | bd |
| Does this have blockers/dependencies? | YES | bd |
| Is this fuzzy/exploratory work? | YES | bd |
| Will this be done in this session? | YES | TodoWrite |
| Is this just a task list for me right now? | YES | TodoWrite |

**Decision Rule**: If resuming in 2 weeks would be hard without bd, use bd.

---

## Conventions

| Convention | Description |
|------------|-------------|
| **Priority** | 0=critical, 1=high, 2=medium, 3=low, 4=backlog |
| **Types** | `bug`, `feature`, `task`, `epic`, `chore` |
| **Epic naming** | Prefix with `Epic: ...` (e.g., `Epic: OAuth Implementation`) |
| **Task naming** | Action-oriented (e.g., "Implement auth endpoints") |
| **IDs** | Format: `<project-slug>-<short-hash>` (e.g., `myproject-abc`) |
| **CLI output** | Always use `--json` for machine-readable output in automation |
| **Detection** | Only use bd when: `which bd` succeeds AND `conductor/beads.json` has `"enabled": true` |

---

## Prerequisites

**Required**:
- **bd CLI**: Version 0.34.0 or later installed and in PATH
- **Git Repository**: Current directory must be a git repo
- **Initialization**: `bd init` must be run once (humans do this, not agents)

**Verify Installation**:
```bash
bd --version  # Should return 0.34.0 or later
```

**Graph Analysis** (optional but recommended):
```bash
bv --version  # Graph validation tool for dependency analysis
```

---

## Quickstart

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `bd ready` | Find tasks with no open blockers |
| 2 | Pick highest priority | Choose P0 > P1 > P2 > P3 > P4 |
| 3 | `bd show <id>` | View details and dependencies |
| 4 | `bd update <id> --status in_progress` | Start working |
| 5 | `bd update <id> --notes "..."` | Add progress notes (critical for compaction survival) |
| 6 | `bd close <id> --reason "..."` | Complete with summary |
| 7 | `bd ready` | Check newly unblocked work |

See detailed sections below for each workflow.

---

## Session Start Protocol

| Aspect | Description |
|--------|-------------|
| **Input** | Existing beads graph in `.beads/issues.jsonl`, active repo |
| **Output** | One task selected, marked `in_progress`, notes started |
| **Done when** | Highest-priority ready task chosen and updated with status + initial notes |

1. **Check for Ready Work**: `bd ready` - shows tasks with no open blockers
2. **Pick Highest Priority**: Choose P0 > P1 > P2 > P3 > P4
3. **Get Full Context**: `bd show <task-id>` - view details and dependencies
4. **Start Working**: `bd update <task-id> --status in_progress`
5. **Add Notes as You Work**: `bd update <task-id> --notes "Progress..."` (critical for compaction survival)

---

## Task Creation

| Aspect | Description |
|--------|-------------|
| **Input** | New piece of work identified, priority and type chosen |
| **Output** | New bead created with title, priority, type (and parent if epic) |
| **Done when** | Task ID returned and linked to epic or dependencies if needed |

```bash
bd create "Task title" -p 1 --type task
```

**Arguments**:
- **Priority**: 0-4 where 0=critical, 1=high, 2=medium, 3=low, 4=backlog
- **Type**: bug, feature, task, epic, chore

**Epic with Children**:
```bash
bd create "Epic: OAuth Implementation" -p 0 --type epic
# Returns: myproject-abc

bd create "Research OAuth providers" -p 1 --parent myproject-abc
bd create "Implement auth endpoints" -p 1 --parent myproject-abc
```

---

## Dependency Management

| Aspect | Description |
|--------|-------------|
| **Input** | Two or more beads whose dependency relationship is known |
| **Output** | Dependencies recorded via `bd dep add` |
| **Done when** | Blocking relationships reflect actual execution order |

```bash
bd dep add <child-id> <parent-id>  # parent blocks child
bd dep list <task-id>              # view dependencies
bd dep tree <task-id>              # show full tree
```

---

## Completion

| Aspect | Description |
|--------|-------------|
| **Input** | Work done, acceptance criteria met |
| **Output** | Bead closed with clear `--reason` summarizing outcome |
| **Done when** | Bead status is "closed" with adequate summary; newly unblocked work identified |

```bash
bd close <task-id> --reason "Completion summary"
bd ready  # Check newly unblocked work
```

---

## Essential Commands

| Command | Purpose |
|---------|---------|
| `bd ready` | Show tasks ready to work on |
| `bd create "Title" -p 1` | Create new task |
| `bd show <id>` | View task details |
| `bd update <id> --status in_progress` | Start working |
| `bd update <id> --notes "Progress"` | Add progress notes |
| `bd close <id> --reason "Done"` | Complete task |
| `bd dep add <child> <parent>` | Add dependency |
| `bd list` | See all tasks |
| `bd search <query>` | Find tasks by keyword |
| `bd sync` | Sync with git remote |

---

## Common Mistakes

| Mistake | Why It's Bad | Fix |
|---------|--------------|-----|
| Not adding notes | Poor compaction resilience; context lost | Add notes after every significant progress |
| Using TodoWrite for multi-session work | Context lost on compaction | Use bd for work spanning sessions |
| Closing without `--reason` | No audit trail; hard to resume | Always provide completion summary |
| Ignoring dependencies | `bd ready` shows wrong tasks | Maintain accurate blocking relationships |
| Not running `bd ready` after close | Miss newly unblocked work | Always check ready after completion |

---

## Conductor Integration

| Aspect | Description |
|--------|-------------|
| **Input** | Conductor track with `beads_epic` in metadata.json, `beads.json` enabled |
| **Output** | Beads epics/tasks mirroring plan.md; `bd ready` for task selection |
| **Done when** | Track's beads epic and subtasks in sync with Conductor plan |

When used with Conductor, Beads provides persistent task memory:

- Each Conductor track becomes a Beads epic (via `beads_epic` in metadata.json)
- Tasks in plan.md sync to Beads subtasks
- `bd ready` helps select next task
- Notes survive context compaction
- Phase dependencies map to Beads graph

**Detection Rule**: Only use `bd` commands automatically when:
1. `which bd` returns a valid path
2. `conductor/beads.json` exists with `"enabled": true`

See [conductor skill](../conductor/SKILL.md) for full integration details.

---

## Relation to Other Beads Skills

This skill is the **foundation** for all beads workflows. Use it for day-to-day multi-session task tracking.

For full feature planning pipelines, use these specialized skills in order:
1. **planning** - Discovery, risk assessment, spike execution
2. **filing-beads** - Convert plans into epics/issues with dependencies
3. **reviewing-beads** - Quality gate for clarity and completeness
4. **orchestrating-beads** - Multi-agent parallel execution

---

## Glossary

| Term | Definition |
|------|------------|
| **Bead** | Generic unit of work in the BD graph |
| **Epic** | Bead with `type=epic` representing a workstream/feature |
| **Issue/Task** | Bead with `type=task\|bug\|feature\|chore` - executable work items |
| **Dependency** | Relationship where one bead blocks another |

---

## Resources

- Full documentation: https://github.com/steveyegge/beads
- CLI reference: [references/CLI_REFERENCE.md](references/CLI_REFERENCE.md)
- Conductor integration: See [conductor skill](../conductor/SKILL.md)
