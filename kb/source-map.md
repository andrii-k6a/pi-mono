# Source Map

This file is a curated navigation map, not a full inventory.

## Primary Execution Path

- `packages/coding-agent/src/main.ts`
  - CLI entrypoint
  - resolves run mode and builds runtime

- `packages/coding-agent/src/core/sdk.ts`
  - composes runtime services
  - creates `Agent` and wraps it in `AgentSession`

- `packages/coding-agent/src/core/agent-session.ts`
  - Pi-specific session runtime
  - prompt handling, persistence, retries, compaction, extension hooks

- `packages/agent/src/agent.ts`
  - stateful wrapper around the lower-level agent loop
  - queue handling and event delivery

- `packages/agent/src/agent-loop.ts`
  - actual turn loop
  - model call -> tool execution -> continuation

- `packages/ai/src/stream.ts`
  - provider dispatch by API type

## Persistence and Runtime State

- `packages/coding-agent/src/core/session-manager.ts`
  - session JSONL storage and tree navigation

- `packages/coding-agent/src/core/settings-manager.ts`
  - merged global/project settings

- `packages/coding-agent/src/core/model-registry.ts`
  - model discovery and auth-backed availability

- `packages/coding-agent/src/core/resource-loader.ts`
  - loads prompts, skills, themes, extensions, and AGENTS files

- `packages/coding-agent/src/core/system-prompt.ts`
  - assembles the effective system prompt

## Provider Layer

- `packages/ai/src/types.ts`
  - core model, context, event, and option types

- `packages/ai/src/api-registry.ts`
  - provider registration and lookup

- `packages/ai/src/providers/register-builtins.ts`
  - lazy builtin provider registration

- `packages/ai/src/providers/openai-responses.ts`
  - useful first provider to study

- `packages/ai/src/providers/anthropic.ts`
  - alternative provider to study for reasoning/tool flow

## Interactive UI

- `packages/coding-agent/src/modes/interactive/`
  - interactive mode shell around `AgentSession`

- `packages/tui/src/tui.ts`
  - differential rendering engine

- `packages/tui/src/components/editor.ts`
  - multiline input editor
