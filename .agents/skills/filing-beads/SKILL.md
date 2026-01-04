---
name: filing-beads
description: Converts feature plans into beads epics and issues with appropriate dependencies and priorities when filing work from a plan, creating new epics, or setting up a project backlog.
---

# Filing Beads from Plan

**Prerequisite**: `beads` skill / `bd` CLI initialized in this repo (see [beads skill](../beads/SKILL.md))

Used in Planning Phase 4: Decomposition. Beads are stored in `.beads/issues.jsonl`. Always use `bd` commands as the source of truth.

## Overview

| Aspect | Description |
|--------|-------------|
| **Input** | Feature plan (e.g., `history/<feature>/discovery.md` and `history/<feature>/approach.md`), plus any spike learnings |
| **Output** | Beads epics and issues in `.beads/issues.jsonl` with priorities and dependencies set |
| **Done when** | All work from plan represented as beads with clear acceptance criteria, sane priorities, and valid DAG with ready work |

## Conventions

- **Epic naming**: Title with `Epic: <title>` for visibility
- **Issue naming**: Action-oriented titles (e.g., "Implement X", "Add Y", "Configure Z")
- **Priority scale**: 0=critical, 1=high, 2=standard, 3=nice-to-have, 4=backlog
- **Dependencies**: Use `--deps bd-<id>` and `--parent` for epic relationships
- **Graph**: Must form valid DAG (no cycles); prefer minimal blocking deps
- **Source of truth**: `.beads/issues.jsonl` managed only via `bd` CLI

---

## Step 1: Understand the Plan

| Aspect | Description |
|--------|-------------|
| **Input** | Plan documents or user-provided context |
| **Output** | Clear understanding of what needs to be filed |
| **Done when** | You can enumerate major workstreams and tasks |

If the `planning` skill was used, start from:
- `history/<feature>/discovery.md` - Codebase snapshot
- `history/<feature>/approach.md` - Strategy and risk map

If no specific plan is provided, ask the user to share the plan or check `history/` directory.

---

## Step 2: Analyze and Structure

| Aspect | Description |
|--------|-------------|
| **Input** | Plan documents from Step 1 |
| **Output** | Draft mapping of plan → epics/issues/deps |
| **Done when** | You have identified all epics, tasks, and their relationships |

Before filing any issues, analyze the plan for:

1. **Major workstreams** - These become epics
2. **Individual tasks** - These become issues under epics
3. **Dependencies** - What must complete before other work can start?
4. **Parallelization opportunities** - What can be worked on simultaneously?
5. **Technical risks** - What needs spikes or investigation first?

---

## Step 3: File Epics First

| Aspect | Description |
|--------|-------------|
| **Input** | List of major workstreams from Step 2 |
| **Output** | Created epics in `.beads/issues.jsonl` |
| **Done when** | Every major workstream has a corresponding epic |

```bash
bd create "Epic: <title>" -t epic -p <priority> --json
```

Epics should:
- Have clear, descriptive titles
- Include acceptance criteria in the description
- Be scoped to deliverable milestones

---

## Step 4: File Detailed Issues

| Aspect | Description |
|--------|-------------|
| **Input** | Epics and detailed plan sections |
| **Output** | Child issues with full metadata |
| **Done when** | Each epic has complete set of child issues covering the plan |

```bash
bd create "<task title>" -t <type> -p <priority> --parent <epic-id> --json
```

Each issue MUST include:
- **Clear title** - Action-oriented (e.g., "Implement X", "Add Y")
- **Detailed description** - What exactly needs to be done
- **Acceptance criteria** - How do we know it's done?
- **Technical notes** - Implementation hints, gotchas, relevant files
- **Dependencies** - Link to blocking issues with `--deps bd-<id>`

---

## Step 5: Map Dependencies Carefully

| Aspect | Description |
|--------|-------------|
| **Input** | All created issues |
| **Output** | Issues linked with correct dependencies |
| **Done when** | All blocking relationships established, graph is DAG |

For each issue, consider:
- Does this depend on another issue completing first?
- Can this be worked on in parallel with siblings?
- Are there cross-epic dependencies?

Use `--deps bd-X,bd-Y` for multiple dependencies.

---

## Step 6: Set Priorities Thoughtfully

| Aspect | Description |
|--------|-------------|
| **Input** | Issues with dependencies |
| **Output** | Issues with appropriate priorities |
| **Done when** | All issues have priorities aligned with execution order |

| Priority | Meaning |
|----------|---------|
| `0` | Critical path blockers, security issues |
| `1` | Core functionality, high business value |
| `2` | Standard work items (default) |
| `3` | Nice-to-haves, polish |
| `4` | Backlog, future considerations |

---

## Step 7: Verify the Graph

| Aspect | Description |
|--------|-------------|
| **Input** | All filed issues |
| **Output** | Validated dependency graph |
| **Done when** | DAG is valid, ready work exists, no gaps |

```bash
bd list --json
bd ready --json
bd dep cycles --json
```

Verify:
- All epics have child issues
- Dependencies form a valid DAG (no cycles)
- Ready work exists (some issues have no blockers)
- Priorities make sense for execution order

---

## Example

See [references/filing-example-user-auth.md](references/filing-example-user-auth.md) for a complete worked example with commands and resulting graph.

---

## Output Format

After completing, provide:

1. Summary of epics created
2. Summary of issues per epic
3. Dependency graph overview (what unblocks what)
4. Suggested starting points (ready issues)
5. Parallelization opportunities

**Next step**: Run `skill("reviewing-beads")` to polish and validate issue quality.
