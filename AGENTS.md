# Agent Routing

## How to give an instruction

### Option 1: Kortix session (right here)
Just talk to me. I'll route to other agents when needed.

### Option 2: GitHub Issue
Open an issue on the target repo with one of these labels:
- `kortix-build` - I (Kortix) handle it directly
- `openhands-task` - Send to OpenHands for full VM work
- `sweep` - Sweep for issue-to-PR
- `morph-edit` - Morph for fleet-wide edits
- `aider-task` - Aider for conversational coding
- `claude-code` - Claude Code CLI

The orchestrator workflow picks it up and routes.

### Option 3: Direct dispatch
```bash
curl -X POST -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/repos/muffy86/agent-factory/dispatches \
  -d '{"event_type":"build_request","client_payload":{"task":"<what>","target_repo":"<where>"}}'
```

## State recovery

State is in `muffy86/.agent-state`. To resume a previous build:
```bash
gh secret list --org Muffy1
cat /tmp/agent-factory/state.json  # or read from .agent-state repo
```
