# Filing Example: User Authentication

This example demonstrates the complete flow of filing beads from a simple plan.

## Input Plan (from discovery)

- Implement user authentication
  - Add login API endpoint
  - Add logout API endpoint
  - Store sessions in Redis

## Filing Commands

### 1. Create Epic

```bash
bd create "User Authentication" -p 1 --type epic
# Returns: myproject-abc
```

### 2. Create Issues with Dependencies

```bash
# Setup Redis first (no dependencies - can start immediately)
bd create "Setup Redis session store" -p 1 --type task --parent myproject-abc
# Returns: myproject-def

# Login depends on Redis being set up
bd create "Implement login endpoint" -p 1 --type task --parent myproject-abc --deps myproject-def
# Returns: myproject-ghi

# Logout also depends on Redis
bd create "Implement logout endpoint" -p 2 --type task --parent myproject-abc --deps myproject-def
# Returns: myproject-jkl
```

## Resulting Graph

```
myproject-abc (Epic: User Authentication)
├── myproject-def (Setup Redis session store) [READY]
├── myproject-ghi (Implement login endpoint) [blocked by def]
└── myproject-jkl (Implement logout endpoint) [blocked by def]
```

## Analysis

| Metric | Value |
|--------|-------|
| Total beads | 4 (1 epic + 3 issues) |
| Ready issues | 1 (myproject-def) |
| Parallelizable after def | 2 (ghi and jkl can run together) |
| Critical path | def → ghi (or def → jkl) |

## Verification Commands

```bash
# List all issues
bd list --json

# Show ready work
bd ready --json

# Verify no cycles
bd dep cycles --json

# Show dependency tree
bd dep tree myproject-abc --json
```
