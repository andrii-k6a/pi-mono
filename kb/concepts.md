# Concepts

## Four-layer mental model

Pi is easiest to reason about in four layers:

1. `packages/ai` — provider abstraction and normalized stream events
2. `packages/agent` — generic agent runtime and turn loop
3. `packages/coding-agent` — Pi-specific sessions, resources, and modes
4. `packages/tui` — terminal UI rendering and input widgets

## Run modes mental model

The same core runtime can be exposed through different shells:

- `interactive` — built-in TUI for humans
- `print` — one-shot human-readable CLI output
- `json` — one-shot structured event stream for scripts and custom UIs
- `rpc` — long-lived JSONL protocol over stdin/stdout for external process integration
- `sdk` — in-process Node/TypeScript embedding

Useful shortcut:
- JSON mode helps you observe one run
- RPC mode lets another program control a live Pi process
- SDK mode lets a Node/TypeScript app call Pi directly as a library

## Session persistence basics

Without `--no-session`, Pi normally persists session history via `SessionManager` into JSONL files under `~/.pi/agent/sessions/`.

Useful distinction:
- new run with sessions enabled -> creates a new persisted session
- continue/resume/open existing session -> appends new entries to an existing session file
- `--no-session` -> no persisted session file

Session history is Pi's own saved conversation/session state, not shell history.

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

## Skills mental model

Skills are prompt-layer resources, not provider-native APIs and not tool contracts.

There are two main paths:
- automatic path: the system prompt advertises skill `name`, `description`, and `location`; the model can then use the `read` tool to load the full `SKILL.md`
- explicit path: `/skill:name` expands before the provider call into a single user message containing a `<skill ...>` block followed by any trailing arguments

Useful ordering details:
- skill metadata XML is appended near the end of the system prompt, after project context files and before the final date/cwd lines
- `/skill:name` puts the full skill block before the user's trailing text inside one user message
- `disable-model-invocation: true` hides a skill from automatic system-prompt advertisement but still allows explicit `/skill:name`

## AGENTS.md mental model

`AGENTS.md`/`CLAUDE.md` are loaded by `resource-loader.ts` as prompt-layer context files.

Traversal/order:
- global agent dir first (`~/.pi/agent/AGENTS.md` or `CLAUDE.md`)
- then walk upward from `cwd` to filesystem root
- per directory, prefer `AGENTS.md` over `CLAUDE.md`
- final order is outermost ancestor to innermost/current directory

They are injected into the system prompt under project context, not treated as a separate runtime mechanism.

## Tmux / background bash philosophy

Pi's default `bash` tool is for bounded command execution whose output becomes model context.

Long-lived or interactive terminal workflows are intentionally delegated to tmux instead of a built-in Pi job-control subsystem.

Useful distinction:
- Pi may still invoke `tmux ...` commands through normal shell execution
- Pi core deliberately avoids first-class built-ins like managed background jobs, pane registries, attach/detach semantics, or hidden long-running process ownership
