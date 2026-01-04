---
name: conductor
description: |
  Context-driven development methodology for organized, spec-first coding. Use when:
  - Project has a `conductor/` directory
  - User mentions specs, plans, tracks, or context-driven development
  - Files like `conductor/tracks.md`, `conductor/product.md`, `conductor/workflow.md` exist
  - User asks about project status, implementation progress, or track management
  - User wants to organize development work with TDD practices
  - User invokes `/conductor-*` commands (setup, newtrack, implement, status, revert, validate, block, skip, revise, archive, export, refresh)
  - User mentions documentation is outdated or wants to sync context with codebase changes
  
  Interoperable with Gemini CLI extension and Claude Code commands.
  Integrates with Beads for persistent task memory across sessions.
---

# Conductor: Context-Driven Development

Measure twice, code once.

## Overview

Conductor enables context-driven development by:
1. Establishing project context (product vision, tech stack, workflow)
2. Organizing work into "tracks" (features, bugs, improvements)
3. Creating specs and phased implementation plans
4. Executing with TDD practices and progress tracking
5. **Multi-agent parallel execution** via orchestrating-beads skill

## Integrated Workflow

Conductor now integrates with the planning skill pipeline for complex features:

```
┌──────────────────────────────────────────────────────────────┐
│                    /conductor-setup                          │
│                    (one-time setup)                          │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ DEEP RESEARCH PHASE (NEW)                           │   │
│   │ • Greenfield: exa-code + Oracle → tech recommendations │
│   │ • Brownfield: gkg repo_map + Oracle → code analysis   │
│   └─────────────────────────────────────────────────────┘   │
│   Output: product.md, tech-stack.md, workflow.md, tracks.md  │
└──────────────────────────────────────────────────────────────┘
                              │
                     PROJECT ĐÃ SẴN SÀNG
                              │
    ══════════════════════════════════════════════════════════
    ║         MỖI KHI CẦN THÊM FEATURE/BUG MỚI               ║
    ║                  (User tự chọn)                         ║
    ══════════════════════════════════════════════════════════
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
    /conductor-newtrack               planning skill
    (classic mode)                    (full pipeline)
    ───────────────                   ───────────────
    • Interactive Q&A                 • Discovery (parallel agents)
    • spec.md + plan.md               • Risk assessment (Oracle)
    • Beads minimal                   • Spikes for HIGH risk
    • Single/few files                • filing-beads → beads graph
                                      • reviewing-beads → quality gate
                                      • orchestrating-beads → execution
```

### When to use which?

| Use Case | Entry Point |
|----------|-------------|
| Bug fix (1-5 files, known location) | `/conductor-newtrack` |
| Small chore/refactor | `/conductor-newtrack` |
| Large feature (multi-module) | `planning` skill |
| External integration (APIs, payments) | `planning` skill |
| Unknown scope (needs discovery) | `planning` skill |

## Context Loading

When this skill activates, load these files to understand the project:
1. `conductor/product.md` - Product vision and goals
2. `conductor/tech-stack.md` - Technology constraints
3. `conductor/workflow.md` - Development methodology (TDD, commits)
4. `conductor/tracks.md` - Current work status

**Important**: Conductor commits locally but never pushes. Users decide when to push to remote.

For active tracks, also load:
- `conductor/tracks/<track_id>/spec.md`
- `conductor/tracks/<track_id>/plan.md`

## Beads Integration

Conductor **always checks for Beads** and offers integration when available. If `bd` CLI is unavailable or disabled, the user can continue without it.

### Detection (MUST check before using bd commands)

Before using ANY `bd` command, you MUST verify:
1. `bd` CLI is installed: `which bd` returns a path
2. `conductor/beads.json` exists AND has `"enabled": true`

```bash
# Check availability - run this before any bd command
if which bd > /dev/null 2>&1 && [ -f conductor/beads.json ]; then
  BEADS_ENABLED=$(cat conductor/beads.json | grep -o '"enabled"[[:space:]]*:[[:space:]]*true' || echo "")
  if [ -n "$BEADS_ENABLED" ]; then
    # Beads is available and enabled - use bd commands
  fi
fi
```

**Detection Rule**: Only use `bd` commands if ALL conditions are met:
1. `which bd` returns a valid path
2. `conductor/beads.json` exists
3. `"enabled": true` in beads.json

### If Beads is NOT available:
- **DO NOT** run any `bd` commands
- Use only plan.md markers for task tracking
- All conductor commands work normally without Beads

### If Beads IS available:
- Tracks become Beads epics
- Tasks sync to Beads for persistent memory
- Use `bd ready` instead of manual task selection
- Notes survive context compaction

## Bead-Backed Tracks

When a track has `beads_epic` in its `metadata.json`:

1. **Task-level tracking** is in Beads (not plan.md checkboxes)
2. **Parallel execution** uses `orchestrating-beads` skill (not Conductor's parallel annotations)
3. **Status** is derived from Beads (`bd list`, `bv --robot-triage`)

### Implementation Detection

```javascript
// In /conductor-implement
const metadata = JSON.parse(fs.readFileSync('conductor/tracks/<id>/metadata.json'));
if (metadata.beads_epic) {
  // Delegate to orchestrating-beads skill
  skill("orchestrating-beads");
} else {
  // Use classic Conductor implementation
}
```

## Proactive Behaviors

1. **On new session**: Check for in-progress tracks, offer to resume
2. **On task completion**: Suggest next task or phase verification
3. **On blocked detection**: Alert user and suggest alternatives
4. **On all tasks complete**: Congratulate and offer archive/cleanup
5. **On stale context detected**: If setup >2 days old or significant codebase changes detected, suggest `/conductor-refresh`
6. **On Beads available**: If `bd` CLI detected during setup, offer integration
7. **On complex feature**: Suggest `planning` skill instead of classic newtrack

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
| "This needs revision" / "Spec is wrong" | `/conductor-revise` |
| "Save context" / "Handoff" | `/conductor-handoff` |
| "Archive completed" | `/conductor-archive` |
| "Export summary" | `/conductor-export` |
| "Docs are outdated" / "Sync with codebase" | `/conductor-refresh` |

## Related Skills

| Skill | Purpose | When to Use |
|-------|---------|-------------|
| **beads** | Foundation skill for bd CLI and task graph | All bead-backed work |
| **planning** | Discovery, risk assessment, spike workflow | Complex features |
| **filing-beads** | Create structured beads with dependencies | After planning |
| **reviewing-beads** | Quality gate for beads | Before execution |
| **orchestrating-beads** | Multi-agent parallel execution | Bead-backed tracks |
| **skill-creator** | Guide for creating new skills | Extending Conductor ecosystem |

## References

- **Detailed workflows**: [references/workflows.md](references/workflows.md) - Step-by-step command execution
- **Directory structure**: [references/structure.md](references/structure.md) - File layout and status markers
- **Beads integration**: [references/beads-integration.md](references/beads-integration.md)
