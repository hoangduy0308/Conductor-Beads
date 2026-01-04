# Spike Workflow

Use this workflow during planning Phase 3 to validate HIGH risk items before full decomposition.

## When to Use

- Planning skill identified HIGH risk components needing spikes
- Multiple spikes can run in parallel
- Each spike is time-boxed (30 min default)

## Step-by-Step Workflow

### 1. Create Spike Beads

```bash
# Create spike epic
bd create "Spike: <question>" -t epic -p 0 --json

# Create spike tasks under the epic
bd create "Spike: Test X" -t task --parent <spike-epic-id> --json
bd create "Spike: Test Y" -t task --parent <spike-epic-id> --json
```

### 2. Get Parallel Plan

```bash
bv --robot-plan 2>/dev/null | jq '.plan.tracks'
```

This shows which spikes can run in parallel.

### 3. Spawn Spike Workers

Use `Task()` for each spike with a time-box:

```python
Task(
    description="Spike: Test X",
    prompt="""
    You are spike worker <AgentName>.
    
    OBJECTIVE: <question to answer>
    TIME BOX: 30 minutes
    
    Write findings to: .spikes/<feature>/<spike-id>/
    
    When done:
    - Close bead with: bd close <id> --reason "YES: <approach>" or "NO: <blocker>"
    - Report via Agent Mail
    """
)
```

### 4. Spike Output Location

Workers write their findings to:

```
.spikes/<feature>/<spike-id>/
├── findings.md       # What was discovered
├── code-samples/     # Any prototype code
└── decision.md       # YES/NO with rationale
```

### 5. Close with Learnings

```bash
# If spike validates the approach
bd close <id> --reason "YES: <validated approach>" --json

# If spike reveals a blocker
bd close <id> --reason "NO: <blocker discovered>" --json
```

## Spike Results Integration

After all spikes complete:
1. Read `.spikes/<feature>/*/decision.md` for each spike
2. Update `history/<feature>/approach.md` with learnings
3. Proceed to decomposition (filing-beads) with validated approach
