# Prompt Flow Notes

Partial but durable understanding of the generic agent runtime and queue semantics.

## Runtime boundary so far

- `AgentSession.prompt()` in `packages/coding-agent` is the app-layer entrypoint.
- It performs app-level preflight before the generic agent runtime: extension command dispatch, extension input interception, skill expansion, prompt-template expansion, streaming queue routing, pending bash flush, model/auth checks, compaction checks, message assembly, and `before_agent_start` extension hooks.
- Slash handling is ordered intentionally: extension commands get first shot, then `/skill:name ...` falls through to skill expansion if no extension command named `skill:name` exists.
- `preflightResult?: (success: boolean) => void` is an internal acceptance/rejection callback for callers such as RPC mode. `true` means the input was accepted or handled during preflight; `false` means preflight threw before entering the main run.
- `AgentSession.prompt()` delegates with `this.agent.prompt(messages)`, then also waits on `waitForRetry()`, so the app-level prompt only settles after any session-triggered auto-retry finishes.
- `Agent` in `packages/agent` owns the live transcript/state, queue objects, event emission, tool scheduling mode, and the bridge into the low-level loop.
- `agent-loop.ts` owns the actual turn loop: prompt injection, assistant streaming, tool execution, steering/follow-up injection, and stopping conditions.

## Turn semantics

A turn is one assistant response plus the tool side effects triggered by that response.

Concretely, one turn can include:
- one assistant message
- any `thinking_*`, `text_*`, and `toolcall_*` streaming events for that message
- all tool executions triggered by tool calls in that message
- the resulting `toolResult` messages

Reasoning/thinking blocks do not create extra turns; they are content blocks inside the same assistant message.

A prompt/run is larger than a single turn:
- one `AgentSession.prompt(...)` call starts one agent run
- that run may contain multiple turns if the assistant calls tools, steering messages arrive, or follow-up messages are injected

## Steering vs follow-up

- `steer`: queued after the current turn finishes, before the next LLM call.
- `followUp`: queued only when the agent has no more tool-call work and no steering messages, i.e. when it would otherwise stop.

Practical meaning:
- steer = change the next assistant step
- follow-up = add a later assistant step after the current chain finishes

If a turn produced tool results and a steering message was queued, the next LLM call sees both the tool results and the injected steering message in transcript order.

## Tool execution scheduling

`ToolExecutionMode` is currently agent-wide, not per-tool:
- `parallel` (default): preflight sibling tool calls sequentially, then execute runnable ones concurrently
- `sequential`: prepare/execute/finalize one tool call at a time

"Preflight" means:
- resolve the tool by name
- optionally normalize raw arguments with `prepareArguments`
- validate arguments against the tool schema
- run `beforeToolCall`
- either produce a runnable prepared call or an immediate error/blocked result

"Allowed tools" in the docs means tool calls that passed preflight. It is not a built-in per-tool concurrency allowlist.

Parallel scheduling does not forbid tool-level serialization. Example: coding-agent file mutation tools serialize same-file writes/edits with `withFileMutationQueue(...)` while still allowing different files to mutate in parallel.

## Bash ordering note

- `recordBashResult()` buffers bash execution messages while the agent is streaming.
- `_flushPendingBashMessages()` writes those buffered messages into agent state and session history before the next normal prompt.
- The reason is message ordering: Pi avoids inserting out-of-band bash messages in the middle of assistant/tool-use ordering.

## Agent state terminology

- `initialState.messages` is only the constructor seed.
- `agent.state.messages` is the live mutable transcript after construction.
- There are not two transcripts; the initial one is copied into runtime state during agent creation.

## Proxy note

`streamProxy()` is for browser apps that keep the agent/runtime in the browser but proxy provider API calls through a backend. It replaces the LLM stream function, not the rest of the agent loop.
