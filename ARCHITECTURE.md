# Agent Factory Architecture

## Component Map

### Agents
| Agent | Invocation | Capability |
|-------|-----------|------------|
| Kortix | The session you're in | Orchestration, planning, general dev, code review |
| GitHub Copilot | @copilot in PR comments | PR review, small fixes |
| Copilot Coding Agent | Auto on Copilot PRs | Multi-file PR generation |
| OpenHands | `openhands` workflow | Full Linux VM, long-running tasks |
| Sweep | `sweep` workflow | Issue-to-PR, doc updates |
| Morph Apply | `morph-edit` workflow | Fleet-wide refactors |
| Aider | `aider-task` workflow | Conversational coding with full repo context |
| Cline | VSCode ext (manual) | Local dev with MCP, file edits |
| Continue | VSCode ext (manual) | Local dev, model-agnostic |
| Claude Code | CLI (manual) | Complex multi-file refactors |

### Workflows (in .github/workflows/)

- `orchestrator.yml` - Main entry point. Listens for build_request, routes to agents.
- `kortix-bridge.yml` - Sends work from issue labels to the right agent.
- `sweep.yml` - Sweep integration
- `morph-edit.yml` - Morph integration
- `aider-task.yml` - Aider integration
- `openhands-task.yml` - OpenHands integration
- `state-sync.yml` - Periodic state sync to .agent-state

### State

`muffy86/.agent-state` (private) - All agent state lives here as JSON files.

### Triggers

- Issue label: routes to the agent
- repository_dispatch event_type: routes by event name
- workflow_dispatch: manual trigger

## Routing Logic

```yaml
# .github/agents.yml
agents:
  kortix-build:
    label: kortix-build
    description: Kortix should handle this
    route: kortix  # in this session
  openhands-task:
    label: openhands-task
    description: Full Linux VM, multi-hour tasks
    route: openhands
  sweep:
    label: sweep
    description: Issue-to-PR automation
    route: sweep
  morph-edit:
    label: morph-edit
    description: Fleet-wide refactors
    route: morph
  aider-task:
    label: aider-task
    description: Conversational coding
    route: aider
  copilot:
    label: copilot
    description: @copilot in PR comments
    route: copilot
  claude-code:
    label: claude-code
    description: Anthropic's CLI agent
    route: claude-code
```

## Build Pipeline (end-to-end)

When a request comes in with `kortix-build` label, the orchestrator:

1. **Spec phase** (5 min)
   - Reads the issue/req
   - Generates a spec in `.agent-factory/specs/<id>.md`
   - Updates `.agent-state/builds.json` with the spec

2. **Plan phase** (5 min)
   - Decomposes the spec into tasks
   - Assigns each task to an agent
   - Stores the plan in `.agent-state/builds.json`

3. **Build phase** (variable)
   - Each task fires in parallel/sequence
   - Agents commit to the target repo
   - State updated as tasks complete

4. **Test phase** (5-10 min)
   - Triggers CI on the target repo
   - Polls for green

5. **Deploy phase** (5 min)
   - Triggers GitHub Pages deploy workflow
   - Verifies deployment

6. **Report phase** (1 min)
   - Posts summary to the original issue
   - Writes state to .agent-state/builds.json

## Free Tier Limits

| Agent | Free tier | Cost after |
|-------|-----------|-----------|
| Kortix | Unlimited (this session) | n/a |
| GitHub Copilot | $10/mo (or included in Pro) | n/a |
| OpenHands (self-host) | Free forever | Server cost only |
| Sweep | Free for public repos | Custom for private |
| Morph | 200 edits/mo | $29/mo for Pro |
| Aider | Unlimited | API costs only |
| Cline | Free | API costs only |
| Continue | Free | API costs only |
| Claude Code | Free with Claude API | API costs only |

**Recommended setup:** OpenHands self-hosted (free) + Aider via Claude Sonnet API (~$0.01/task) + everything else free.

## Security

- Each agent has its own PAT scoped to specific repos
- A2A bridge uses short-lived installation tokens (after GitHub App install)
- All state writes go through authenticated workflows
- The Kortix session token is in `.kortix/secrets/github.env` (gitignored)
