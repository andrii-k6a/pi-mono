# Current State

## Current Objective
Build native understanding of Pi internals and keep that understanding recoverable across fresh Pi sessions.

## Current Focus
- Trace the runtime boundary from `AgentSession.prompt()` into `Agent.prompt()` and `runAgentLoop()`
- Finish the README orientation pass with `packages/ai/README.md`, then `packages/tui/README.md`
- Keep durable runtime notes in `kb/artifacts/` and keep `kb/state.md` compact

## What I Understand
- Pi is easiest to understand in four layers: `packages/ai`, `packages/agent`, `packages/coding-agent`, `packages/tui`
- `packages/coding-agent/README.md` is the best first detailed entry point because it describes Pi as the actual app/runtime
- Run modes are different shells around the same core runtime: interactive, print, JSON, RPC, SDK
- Sessions are persisted as JSONL trees; non-message state changes like model and thinking-level updates use dedicated entry types
- Skills are prompt-layer resources, not provider-native APIs or tool contracts
- `AGENTS.md`/`CLAUDE.md` files are loaded from the global agent dir first, then by walking from `cwd` up to filesystem root, preferring `AGENTS.md` over `CLAUDE.md` per directory and ordering outermost to innermost
- In the generic agent runtime, a turn is one assistant response plus the tool side effects caused by that response; thinking blocks stay inside that same turn
- `steer` injects after the current turn and before the next LLM call; `followUp` waits until the agent would otherwise stop
- Tool execution policy is agent-wide (`parallel` by default, `sequential` as a safer fallback); tool implementations can still serialize specific shared resources such as same-file mutations
- `initialState.messages` seeds the runtime transcript; `agent.state.messages` is the live mutable transcript after construction

## What Is Still Fuzzy
- Exact responsibility split between `AgentSession`, `Agent`, and `agent-loop` in a full traced prompt path
- Exact session JSONL entry model and tree branching behavior in code
- Exact provider stream normalization path inside a concrete provider implementation
- Exact mode selection path from CLI entrypoint into interactive/print/JSON/RPC
- Exact ordering between retries, compaction checks, and final session persistence in `AgentSession`

## Important Files
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/agent/README.md`
- `packages/agent/src/agent.ts`
- `packages/agent/src/agent-loop.ts`
- `packages/agent/src/types.ts`
- `kb/artifacts/prompt-flow.md`

## Next Steps
1. Trace `packages/coding-agent/src/core/agent-session.ts::prompt()`
2. Then trace `packages/agent/src/agent.ts::{prompt, runPromptMessages, createContextSnapshot, createLoopConfig}`
3. Then trace `packages/agent/src/agent-loop.ts::{runAgentLoop, runLoop, streamAssistantResponse}`
4. After that runtime-boundary trace, read `packages/ai/README.md`
5. Then read `packages/tui/README.md`

## Open Questions
- What exactly does `AgentSession.prompt()` add before delegating to `Agent.prompt()`?
- Which responsibilities stay in `Agent` versus `agent-loop`?
- Where in `packages/ai` are provider-native stream events normalized into Pi's `AssistantMessageEvent` stream?
- How does `main.ts` choose interactive vs print vs JSON vs RPC concretely?
- What is the exact retry/compaction/session-persistence ordering around failures and retries?

## Pending Artifacts
- `kb/artifacts/session-model.md`

## Resume Protocol
1. Read this file first.
2. Then read `kb/README.md`.
3. Then read files referenced under Current Focus, Important Files, Open Questions, and Pending Artifacts if they exist.
4. Summarize state in 5-10 bullets.
5. Propose one next concrete learning step.
6. Stay in interactive guide mode until asked to close out.