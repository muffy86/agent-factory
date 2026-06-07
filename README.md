# Kortix Agent Factory

**The unified agent team for muffy86's repos.** Coordinates multiple specialized AI agents, each handling what they do best.

## The Team

| Agent | Role | When to use | Status |
|-------|------|-------------|--------|
| **Kortix** (this workspace) | Orchestrator + General dev | "Build me X" / "Fix Y" / "Review Z" | :white_check_mark: Live |
| **GitHub Copilot Coding Agent** | Multi-file PRs in web UI | Use the [Copilot chat](https://github.com/copilot) | :white_check_mark: Live |
| **OpenHands** (All-Hands-AI) | Autonomous dev in cloud VM | Long-running tasks that need a full Linux env | :white_check_mark: Integrated |
| **Sweep** | Issue-to-PR automation | "Make PRs from issues" | :white_check_mark: Integrated |
| **Morph Apply** | AI code edits in workflows | "Refactor X to Y across the fleet" | :white_check_mark: Integrated |
| **Aider** | Chat-with-repo in CLI/GH Actions | "Add tests for module Z" | :white_check_mark: Integrated |
| **Cline** | VSCode/jetbrains agent | Local dev with full repo context | :white_check_mark: Integrated |
| **Continue** | VSCode/jetbrains open-source | Local dev, model-agnostic | :white_check_mark: Integrated |
| **Claude Code** | Anthropic's CLI agent | Complex refactors, multi-file | :white_check_mark: Integrated |

## How to give instructions

Three ways to kick off the team:

### 1. From the Kortix session (right here)
Just say "build X" or "fix Y". The agent orchestrates other agents via the A2A bridge.

### 2. From any GitHub Issue
Tag the issue with one of:
- `kortix-build` - Kortix should build this
- `openhands-task` - Send to OpenHands
- `sweep` - Send to Sweep
- `morph-edit` - Send to Morph for a code edit
- `aider-task` - Send to Aider

### 3. Via the A2A bridge (programmatic)
```bash
curl -X POST -H "Authorization: Bearer $GITHUB_TOKEN" \
  https://api.github.com/repos/muffy86/agent-factory/dispatches \
  -d '{"event_type":"build_request","client_payload":{"task":"ship a CLI todo app","target_repo":"muffy86/new-app"}}'
```

## Persisted state

Every agent writes its state to `muffy86/.agent-state` (private repo). The state files are:

- `state/kortix.json` - Kortix session state, history
- `state/openhands.json` - OpenHands task queue
- `state/sweep.json` - Sweep PR queue
- `state/morph.json` - Morph edit history
- `state/aider.json` - Aider task history
- `state/builds.json` - End-to-end build pipeline state

## Architecture

```
                  ┌─────────────────┐
                  │  Kortix session │  (you are here)
                  │   (this chat)   │
                  └────────┬────────┘
                           │ (dispatches)
                           ▼
                  ┌─────────────────┐
                  │  agent-factory  │  (this repo)
                  │  orchestrator   │
                  └────────┬────────┘
                           │
        ┌──────────┬───────┼───────┬──────────┐
        ▼          ▼       ▼       ▼          ▼
    ┌─────┐   ┌─────┐ ┌─────┐ ┌─────┐   ┌─────┐
    │ Co- │   │Open-│ │Sweep│ │Morph│   │Aider│
    │pilot│   │Hands│ │     │ │     │   │     │
    └─────┘   └─────┘ └─────┘ └─────┘   └─────┘
        │          │       │       │          │
        └──────────┴───────┴───────┴──────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  .agent-state   │  (persisted)
                  │   (private)     │
                  └─────────────────┘
```

## Build pipeline (the "ship it" workflow)

1. **Spec** - Kortix reads the request, writes a spec
2. **Plan** - Kortix breaks the work into tasks, assigns to agents
3. **Build** - Each agent executes in parallel/sequence
4. **Test** - CI runs on the target repo
5. **Deploy** - GitHub Pages or Cloudflare Pages
6. **Report** - State written to .agent-state, summary posted to issue

## State recovery

Every session start, the orchestrator reads `.agent-state` to resume where the last session left off. This is the persisted second-brain.

## Free as in beer

All agents in this factory are free/open-source:
- Kortix: open agent (this workspace)
- OpenHands: Apache 2.0, self-hostable
- Sweep: free for public repos
- Morph: free tier (200 edits/month)
- Aider: MIT, runs anywhere
- Cline: Apache 2.0
- Continue: Apache 2.0
- Claude Code: included with Anthropic API
