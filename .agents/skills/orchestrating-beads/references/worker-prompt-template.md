# Worker Prompt Template

Use this template when spawning worker subagents in orchestrating-beads.

## Template

```
You are agent <AGENT_NAME> working on Track <TRACK_NUMBER> of epic <EPIC_ID>.

## Setup
1. Read the project's AGENTS.md for tool preferences
2. Register yourself with Agent Mail

## Your Track
**Beads to complete IN ORDER**: <BEAD_LIST>
**File scope**: <FILE_SCOPE>

## Protocol for EACH bead:

### Start Bead
1. Register: 
   ```
   register_agent(
     project_key="<PROJECT_PATH>",
     name="<AGENT_NAME>",
     program="amp",
     model="<MODEL>",
     task_description="Working on <bead-id>"
   )
   ```

2. Read context from previous bead:
   ```
   summarize_thread(
     project_key="<PROJECT_PATH>",
     thread_id="track:<AGENT_NAME>:<EPIC_ID>"
   )
   ```

3. Reserve your files:
   ```
   file_reservation_paths(
     project_key="<PROJECT_PATH>",
     agent_name="<AGENT_NAME>",
     paths=["<FILE_SCOPE>"],
     reason="Working on <bead-id>",
     ttl_seconds=3600
   )
   ```

4. Claim the bead:
   ```bash
   bd update <bead-id> --status in_progress
   ```

### Work on Bead
- Read the bead details: `bd show <bead-id>`
- Implement according to acceptance criteria
- Run tests if applicable
- Check inbox periodically for messages from orchestrator:
  ```
  fetch_inbox(
    project_key="<PROJECT_PATH>",
    agent_name="<AGENT_NAME>",
    limit=5
  )
  ```
- If blocked, send high-priority message:
  ```
  send_message(
    project_key="<PROJECT_PATH>",
    sender_name="<AGENT_NAME>",
    to=["<ORCHESTRATOR_NAME>"],
    thread_id="<EPIC_ID>",
    subject="[<bead-id>] BLOCKED",
    body_md="Blocked by: <reason>. Need: <what you need>",
    importance="high"
  )
  ```

### Complete Bead
1. Close the bead:
   ```bash
   bd close <bead-id> --reason "Summary of what was done"
   ```

2. Report to orchestrator:
   ```
   send_message(
     project_key="<PROJECT_PATH>",
     sender_name="<AGENT_NAME>",
     to=["<ORCHESTRATOR_NAME>"],
     thread_id="<EPIC_ID>",
     subject="[<bead-id>] COMPLETE",
     body_md="Done: <summary>. Commits: <sha>. Next: <next-bead-id>"
   )
   ```

3. Save context for next bead (or next session):
   ```
   send_message(
     project_key="<PROJECT_PATH>",
     sender_name="<AGENT_NAME>",
     to=["<AGENT_NAME>"],
     thread_id="track:<AGENT_NAME>:<EPIC_ID>",
     subject="<bead-id> Complete - Context",
     body_md="""
## What I Did
- <summary>

## Key Learnings
- <insight 1>
- <insight 2>

## Gotchas for Next Bead
- <warning 1>

## Next Bead Notes
- <preparation for next>
"""
   )
   ```

4. Release file reservations:
   ```
   release_file_reservations(
     project_key="<PROJECT_PATH>",
     agent_name="<AGENT_NAME>"
   )
   ```

### Continue to Next Bead
- Loop back to "Start Bead" with the next bead in your track
- Read your track thread for context from previous bead

## When Track Complete

Send final report:
```
send_message(
  project_key="<PROJECT_PATH>",
  sender_name="<AGENT_NAME>",
  to=["<ORCHESTRATOR_NAME>"],
  thread_id="<EPIC_ID>",
  subject="[Track <TRACK_NUMBER>] COMPLETE",
  body_md="""
## Track Complete: <AGENT_NAME>

### Beads Completed
- <bead-1>: <summary>
- <bead-2>: <summary>
- <bead-3>: <summary>

### Commits
- <sha1>: <message>
- <sha2>: <message>

### Key Learnings
- <insight>

### Any Issues for Other Tracks
- <cross-track notes if any>
"""
)
```

Return a summary of all work completed for the orchestrator.
```

## Placeholders

Replace these when generating the actual prompt:

| Placeholder | Example |
|-------------|---------|
| `<AGENT_NAME>` | BlueLake |
| `<TRACK_NUMBER>` | 1 |
| `<EPIC_ID>` | bd-abc123 |
| `<BEAD_LIST>` | bd-1, bd-2, bd-3 |
| `<FILE_SCOPE>` | packages/sdk/** |
| `<PROJECT_PATH>` | /path/to/project |
| `<ORCHESTRATOR_NAME>` | PurpleConductor |
| `<MODEL>` | claude-sonnet-4-20250514 |
