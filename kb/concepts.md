# Concepts

## Four-layer mental model

Pi is easiest to reason about in four layers:

1. `packages/ai` — provider abstraction and normalized stream events
2. `packages/agent` — generic agent runtime and turn loop
3. `packages/coding-agent` — Pi-specific sessions, resources, and modes
4. `packages/tui` — terminal UI rendering and input widgets

## AgentSession vs Agent

Working hypothesis:
- `AgentSession` is Pi policy and session/runtime orchestration
- `Agent` is reusable runtime machinery around the lower-level loop

This distinction still needs deeper validation by tracing `prompt()` end to end.

## Model vs provider vs API

- provider: human-facing source like OpenAI, Anthropic, Google
- API: protocol implementation such as `openai-responses`
- model: concrete model metadata plus capability flags

Useful shortcut:
- model selection picks metadata
- stream dispatch picks implementation
