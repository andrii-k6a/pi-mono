# Current State

## Current Objective
Build native understanding of Pi internals and keep that understanding recoverable across fresh Pi sessions.

## Current Focus
- Establish a low-overhead study workflow in `kb/`
- Restore study state through `/study`
- Use the Pi internals guide as the main reading plan

## What I Understand
- Pi is easiest to understand in four layers: `packages/ai`, `packages/agent`, `packages/coding-agent`, `packages/tui`
- `packages/coding-agent/src/main.ts` is the CLI/runtime entrypoint
- `packages/coding-agent/src/core/sdk.ts` composes the runtime and creates `AgentSession`
- `AgentSession` is the Pi-specific runtime shell around the generic `Agent`
- `packages/agent/src/agent.ts` wraps the lower-level agent loop with state and events
- `packages/ai/src/stream.ts` dispatches to the provider implementation selected by model API

## What Is Still Fuzzy
- Exact responsibility split between `AgentSession`, `Agent`, and `agent-loop`
- Exact session JSONL entry model and tree branching behavior
- Exact provider stream normalization path inside a concrete provider implementation

## Important Files
- `kb/pi-internals-guide.md`
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/agent/src/agent.ts`
- `packages/agent/src/agent-loop.ts`
- `packages/ai/src/stream.ts`

## Next Steps
1. Run `/study` at the beginning of each fresh session
2. Trace `AgentSession.prompt()` into `Agent.runPromptMessages()` and the loop
3. Capture the first end-to-end prompt flow artifact in `kb/artifacts/`

## Open Questions
- Where are non-message session state changes persisted?
- When do retries happen relative to compaction?
- Where exactly are provider stream events translated into shared assistant events?

## Pending Artifacts
- `kb/artifacts/prompt-flow.md`
- `kb/artifacts/session-model.md`

## Resume Protocol
1. Read this file first.
2. Then read `kb/README.md`.
3. Then read files referenced under Current Focus, Important Files, Open Questions, and Pending Artifacts if they exist.
4. Summarize state in 5-10 bullets.
5. Propose one next concrete learning step.
6. Stay in interactive guide mode until asked to close out.
