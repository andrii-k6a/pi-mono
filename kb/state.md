# Current State

## Current Objective
Build native understanding of Pi internals and keep that understanding recoverable across fresh Pi sessions.

## Current Focus
- Finish the package README orientation pass
- Keep the `kb/` workflow lightweight and recoverable through `/study`
- Build clean mental models for runtime boundaries and prompt-layer resources (`skills`, `AGENTS.md`, system prompt assembly)

## What I Understand
- Pi is easiest to understand in four layers: `packages/ai`, `packages/agent`, `packages/coding-agent`, `packages/tui`
- `packages/coding-agent/README.md` is the best first detailed entry point because it describes Pi as the actual app/runtime
- Run modes are different shells around the same core runtime: interactive, print, JSON, RPC, SDK
- Sessions are persisted as JSONL trees; non-message state changes like model and thinking-level updates use dedicated entry types
- Skills are prompt-layer resources, not provider-native APIs or tool contracts
- Automatic skill use works by advertising skill metadata in the system prompt and letting the model `read` the full `SKILL.md`; `/skill:name` injects the full skill body into the user message before trailing args
- `AGENTS.md`/`CLAUDE.md` files are loaded from the global agent dir first, then by walking from `cwd` up to filesystem root, preferring `AGENTS.md` over `CLAUDE.md` per directory and ordering outermost to innermost
- Pi's “no background bash” philosophy means Pi avoids a built-in job-control subsystem; tmux remains the substrate for long-lived or interactive shell workflows, even if Pi can invoke tmux commands

## What Is Still Fuzzy
- Exact responsibility split between `AgentSession`, `Agent`, and `agent-loop`
- Exact session JSONL entry model and tree branching behavior in code
- Exact provider stream normalization path inside a concrete provider implementation
- Exact mode selection path from CLI entrypoint into interactive/print/JSON/RPC

## Important Files
- `packages/coding-agent/README.md`
- `packages/coding-agent/docs/json.md`
- `packages/coding-agent/docs/rpc.md`
- `packages/coding-agent/docs/sdk.md`
- `packages/coding-agent/docs/session.md`
- `packages/coding-agent/docs/skills.md`
- `packages/coding-agent/docs/tmux.md`
- `packages/coding-agent/src/core/resource-loader.ts`
- `packages/coding-agent/src/core/system-prompt.ts`
- `packages/coding-agent/src/core/skills.ts`

## Next Steps
1. Read `packages/agent/README.md`
2. Then read `packages/ai/README.md`
3. Then read `packages/tui/README.md`
4. After the README pass, trace `AgentSession.prompt()` into `Agent.runPromptMessages()` and `agent-loop.ts`

## Open Questions
- What is the precise responsibility boundary between `AgentSession`, `Agent`, and `agent-loop`?
- In concrete provider code, where are raw provider stream events translated into Pi's normalized assistant event stream?
- What is the exact ordering between retries, compaction checks, and final session persistence?
- How does `main.ts` route into each run mode concretely?

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
