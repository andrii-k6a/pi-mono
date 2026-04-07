# Pi Internals Guide

This guide is for building a working mental model of Pi internals in the `pi-mono` repository.

The goal is not to memorize every subsystem. The goal is to understand:

1. where a prompt enters the system
2. how Pi chooses models and tools
3. how the agent loop runs
4. how provider streams are normalized
5. how sessions are persisted
6. how the TUI sits on top of the runtime
7. where extensions hook into the flow

---

## The Big Picture

Pi is easiest to understand as four layers:

1. **`packages/ai`**
   - model registry and provider abstraction
   - streaming requests to OpenAI, Anthropic, Google, etc.
   - normalizing provider output into a shared event model

2. **`packages/agent`**
   - generic stateful agent runtime
   - turn loop
   - tool execution
   - steering and follow-up queues
   - event emission

3. **`packages/coding-agent`**
   - Pi-specific runtime
   - sessions, compaction, model selection, prompt templates, skills, extensions
   - interactive mode, print mode, RPC mode, SDK

4. **`packages/tui`**
   - terminal rendering
   - editor and widgets
   - overlays and input handling

Keep that separation in mind while reading. Most confusion comes from mixing these layers together.

---

## Recommended Reading Order

Read in this order.

### Pass 1: architecture and main execution path

1. `README.md`
2. `packages/coding-agent/README.md`
3. `packages/agent/README.md`
4. `packages/ai/README.md`
5. `packages/tui/README.md`
6. `packages/coding-agent/src/main.ts`
7. `packages/coding-agent/src/core/sdk.ts`
8. `packages/coding-agent/src/core/agent-session.ts`
9. `packages/agent/src/agent.ts`
10. `packages/agent/src/agent-loop.ts`
11. `packages/ai/src/stream.ts`

At the end of Pass 1, you should be able to answer:
- where does a prompt start?
- what object owns session state?
- what object owns the actual agent loop?
- where do model provider calls happen?

### Pass 2: persistence and context management

1. `packages/coding-agent/src/core/session-manager.ts`
2. `packages/coding-agent/src/core/system-prompt.ts`
3. `packages/coding-agent/src/core/resource-loader.ts`
4. `packages/coding-agent/src/core/settings-manager.ts`
5. `packages/coding-agent/src/core/model-registry.ts`
6. `packages/coding-agent/src/core/model-resolver.ts`
7. `packages/coding-agent/docs/session.md`
8. `packages/coding-agent/docs/compaction.md`

At the end of Pass 2, you should understand:
- how sessions are stored
- how Pi restores models and thinking levels
- where prompt templates, skills, AGENTS files, and extensions come from
- how compaction fits into long conversations

### Pass 3: provider and tool execution internals

1. `packages/ai/src/types.ts`
2. `packages/ai/src/api-registry.ts`
3. `packages/ai/src/providers/register-builtins.ts`
4. one provider file:
   - `packages/ai/src/providers/openai-responses.ts`, or
   - `packages/ai/src/providers/anthropic.ts`
5. `packages/ai/src/providers/transform-messages.ts`
6. `packages/ai/src/utils/event-stream.ts`
7. `packages/coding-agent/src/core/tools/index.ts`
8. one built-in tool implementation from `packages/coding-agent/src/core/tools/`

At the end of Pass 3, you should understand:
- how model metadata maps to a provider implementation
- how provider-specific payloads get built
- how provider streams turn into shared assistant events
- how tools are exposed to the model and executed

### Pass 4: interactive mode and UI

1. `packages/coding-agent/src/modes/interactive/*`
2. `packages/tui/src/tui.ts`
3. `packages/tui/src/components/editor.ts`
4. `packages/tui/src/keys.ts`
5. `packages/tui/src/keybindings.ts`
6. `packages/coding-agent/docs/tui.md`
7. `packages/coding-agent/docs/keybindings.md`

At the end of Pass 4, you should understand:
- how the TUI renders without flicker
- how editor input becomes prompts
- how message updates and tool output reach the screen

### Pass 5: extensibility

1. `packages/coding-agent/docs/extensions.md`
2. `packages/coding-agent/docs/sdk.md`
3. `packages/coding-agent/src/core/extensions/*`
4. `packages/coding-agent/examples/extensions/`
5. `packages/coding-agent/examples/sdk/`

At the end of Pass 5, you should understand:
- how extensions intercept tool calls and context
- how custom tools and commands are registered
- how to embed Pi with the SDK

---

## The Most Important Trace

If you only do one exercise, trace this exact scenario:

> User types a prompt in interactive mode. The model calls `read`. Pi executes the tool. The model continues and returns a final answer.

Follow it through these files:

1. `packages/coding-agent/src/modes/interactive/...`
   - user submits text
2. `packages/coding-agent/src/core/agent-session.ts`
   - `prompt()` prepares and validates the turn
3. `packages/agent/src/agent.ts`
   - stateful wrapper around the loop
4. `packages/agent/src/agent-loop.ts`
   - actual LLM/tool/LLM control flow
5. `packages/ai/src/stream.ts`
   - dispatches to the provider
6. provider file in `packages/ai/src/providers/`
   - payload construction + response parsing
7. back to `packages/agent/src/agent.ts`
   - tool execution lifecycle events
8. back to `packages/coding-agent/src/core/agent-session.ts`
   - message persistence and UI-facing events
9. back to interactive mode / TUI
   - final rendering

If you can explain that trace from memory, you understand Pi far better than by reading features in isolation.

---

## Core Runtime Map

### 1. CLI and mode selection

**File:** `packages/coding-agent/src/main.ts`

What it does:
- parses CLI args
- decides interactive vs print vs JSON vs RPC
- creates runtime services
- creates the session runtime
- starts the selected mode

What to look for:
- `resolveAppMode()`
- `createAgentSessionRuntime(...)`
- `InteractiveMode`, `runPrintMode`, `runRpcMode`

Mental model:
- `main.ts` is orchestration only
- it should not contain agent logic or provider logic

### 2. Session construction

**File:** `packages/coding-agent/src/core/sdk.ts`

What it does:
- creates `AuthStorage`, `ModelRegistry`, `SettingsManager`, `SessionManager`
- loads resources
- restores previous session state if present
- creates `Agent`
- wraps it in `AgentSession`

What to look for:
- `createAgentSession()`
- how model restoration works
- how `streamSimple()` is used through the `Agent`
- how built-in tools are selected

Mental model:
- `sdk.ts` is the composition root for the reusable runtime

### 3. Pi-specific session runtime

**File:** `packages/coding-agent/src/core/agent-session.ts`

This is the most important Pi file.

What it does:
- provides the public session abstraction used by all run modes
- turns user input into agent prompts
- expands prompt templates and skill commands
- manages queued steering and follow-up messages
- persists messages to the session file
- handles model switching and thinking level changes
- invokes extensions around tool calls and context
- triggers compaction and retries

What to look for:
- `prompt()`
- `_processAgentEvent()`
- `_emitExtensionEvent()`
- `setModel()`
- `setThinkingLevel()`
- queue handling
- compaction and retry logic

Mental model:
- `AgentSession` is the Pi runtime shell around the generic `Agent`
- if you want to understand “Pi behavior”, this file matters more than `main.ts`

### 4. Generic agent wrapper

**File:** `packages/agent/src/agent.ts`

What it does:
- owns the current `AgentState`
- exposes `prompt()`, `continue()`, `steer()`, `followUp()`
- creates loop config
- runs the agent loop
- updates local state from loop events
- emits ordered events to listeners

What to look for:
- `runPromptMessages()`
- `createLoopConfig()`
- `processEvents()`

Mental model:
- `Agent` is not Pi-specific
- it is the reusable stateful shell around the lower-level loop

### 5. The actual loop

**File:** `packages/agent/src/agent-loop.ts`

What it does:
- runs one or more turns
- calls the model
- detects tool calls
- validates and executes tools
- appends tool results
- continues until the model stops

Mental model:
- this file is where “agent behavior” actually happens
- if Pi feels autonomous, this loop is why

### 6. Provider dispatch

**File:** `packages/ai/src/stream.ts`

What it does:
- resolves the provider implementation by API type
- forwards to `stream()` or `streamSimple()` on that provider

Mental model:
- this file is intentionally thin
- the real provider complexity lives in `packages/ai/src/providers/`

---

## Provider Layer Map

### Key files

- `packages/ai/src/types.ts`
- `packages/ai/src/api-registry.ts`
- `packages/ai/src/providers/register-builtins.ts`
- `packages/ai/src/providers/*.ts`
- `packages/ai/src/providers/transform-messages.ts`

### What to understand

#### Model vs provider vs API

- **provider**: anthropic, openai, google, etc.
- **API**: the protocol shape Pi uses to talk to a provider, e.g. `openai-responses`
- **model**: a concrete model id plus capability metadata

One useful mental shortcut:
- model selection picks **metadata**
- stream dispatch picks **implementation**

#### Provider job

Each provider implementation is responsible for:
- converting Pi context to provider payloads
- converting tool schemas to provider tool formats
- parsing provider streaming output
- emitting normalized events like text deltas, thinking deltas, tool call deltas, usage, and stop reasons

### Recommended first provider

Read `packages/ai/src/providers/openai-responses.ts` first if you want the broadest relevance.

Read `packages/ai/src/providers/anthropic.ts` first if you want a cleaner mental model of reasoning/tool flows.

---

## Session and Persistence Map

### Key files

- `packages/coding-agent/src/core/session-manager.ts`
- `packages/coding-agent/docs/session.md`
- `packages/coding-agent/docs/compaction.md`

### What to understand

Pi stores sessions as JSONL with tree structure.

That explains:
- `/tree`
- `/fork`
- restoring history
- branch summaries
- compaction checkpoints

Important idea:
- Pi does not think of a session as a flat chat log
- it thinks of it as a navigable conversation tree

### Questions to answer while reading

- what is an entry vs a message?
- how are `id` and `parentId` used?
- how does Pi restore the current branch?
- where are model changes and thinking-level changes persisted?

---

## Resource Loading Map

### Key files

- `packages/coding-agent/src/core/resource-loader.ts`
- `packages/coding-agent/src/core/system-prompt.ts`
- `packages/coding-agent/src/core/prompt-templates.ts`
- `packages/coding-agent/src/core/skills.ts`

### What to understand

Pi assembles runtime behavior from discovered resources:
- AGENTS files
- prompt templates
- skills
- themes
- extensions

Important idea:
- the system prompt is not a fixed constant
- it is built from runtime state and discovered resources

This is one of the main reasons Pi feels “open” rather than monolithic.

---

## Interactive Mode Map

### Key files

- `packages/coding-agent/src/modes/interactive/*`
- `packages/tui/src/tui.ts`
- `packages/tui/src/components/editor.ts`

### What to understand

The TUI is a consumer of runtime events. It is not the runtime itself.

The most important split:
- `AgentSession` owns behavior
- `InteractiveMode` owns presentation and interaction
- `TUI` owns rendering mechanics

### `packages/tui/src/tui.ts`

This file explains:
- differential rendering
- overlays
- focus
- hardware cursor positioning for IME
- full redraw vs partial redraw decisions

### `packages/tui/src/components/editor.ts`

This file explains:
- multiline editing
- autocomplete
- cursor behavior
- keyboard input handling

---

## Extension Model Map

### Key files

- `packages/coding-agent/docs/extensions.md`
- `packages/coding-agent/src/core/extensions/*`
- `packages/coding-agent/examples/extensions/`

### What to understand

Extensions are not a bolt-on afterthought. They are part of Pi’s core architecture.

They can:
- register tools
- register slash commands
- intercept tool calls
- modify tool results
- rewrite context
- add UI
- replace provider config

Important idea:
- Pi intentionally keeps many workflow features out of the hardcoded core
- the extension system is how Pi stays small without becoming rigid

---

## Suggested Study Sessions

### Session 1: Build the top-level map
Time: 45–60 min

Read:
- `README.md`
- package READMEs

Write down:
- what each package is responsible for
- where the boundaries are

### Session 2: Trace startup
Time: 60–90 min

Read:
- `packages/coding-agent/src/main.ts`
- `packages/coding-agent/src/core/sdk.ts`

Write down:
- how a session is created
- how model/auth/settings/resource/session services are wired

### Session 3: Trace one prompt
Time: 90 min

Read:
- `packages/coding-agent/src/core/agent-session.ts`
- `packages/agent/src/agent.ts`
- `packages/agent/src/agent-loop.ts`

Write down:
- what happens before the first provider call
- what events are emitted
- where tool calls are executed

### Session 4: Trace one provider
Time: 60–90 min

Read:
- `packages/ai/src/stream.ts`
- one provider file

Write down:
- how Pi converts generic context to provider payloads
- how provider stream output becomes normalized assistant events

### Session 5: Understand persistence
Time: 60 min

Read:
- `packages/coding-agent/src/core/session-manager.ts`
- session docs

Write down:
- how branches are represented
- how Pi resumes from disk

### Session 6: Understand the UI shell
Time: 60–90 min

Read:
- `packages/tui/src/tui.ts`
- editor component
- interactive mode files

Write down:
- how input moves from terminal to session prompt
- how output moves from events to rendered lines

### Session 7: Understand extensibility
Time: 60–90 min

Read:
- extensions docs
- 2–3 example extensions
- SDK docs

Write down:
- what Pi exposes for customization
- which features are policy vs mechanism

---

## High-Value Questions To Answer Yourself

Use these as checkpoints.

1. Why is `AgentSession` separate from `Agent`?
2. Why is `stream.ts` so small?
3. Where does tool execution really happen?
4. Where are queued user messages stored while the agent is still running?
5. How does Pi rebuild the system prompt each turn?
6. How does Pi persist non-message session state like model changes?
7. Why does the TUI not own business logic?
8. Why are extensions able to intercept so much behavior?

If you can answer those clearly, you have a native understanding of the architecture.

---

## What To Ignore At First

Skip these until the core flow is clear:
- package installation and package manager internals
- OAuth implementation details
- migrations
- platform-specific clipboard/image utilities
- Bun entrypoints
- provider edge-case compatibility flags

These are real internals, but not the shortest path to understanding the design.

---

## Practical Exercises

### Exercise 1: Annotate one prompt flow
Create a note with these headings and fill them in:
- input received
- session preprocessing
- agent start
- provider request
- tool call
- tool result
- continuation
- persistence
- UI update

### Exercise 2: Add temporary logging
Add logging in these places on a throwaway branch:
- `AgentSession.prompt()`
- `Agent.processEvents()`
- one provider stream parser

Then run Pi and watch one request end to end.

### Exercise 3: Read a session file
Open one JSONL session file and map entries back to runtime events.

### Exercise 4: Build a minimal extension
Create an extension that:
- logs `tool_call`
- blocks one command pattern
- adds one custom slash command

That will make the extension architecture feel concrete.

---

## Fast Reference: If You Forget Where Something Lives

### “Where does Pi decide what mode to run?”
- `packages/coding-agent/src/main.ts`

### “Where is the session abstraction?”
- `packages/coding-agent/src/core/agent-session.ts`

### “Where is the generic agent runtime?”
- `packages/agent/src/agent.ts`

### “Where is the turn loop?”
- `packages/agent/src/agent-loop.ts`

### “Where do provider requests start?”
- `packages/ai/src/stream.ts`

### “Where do provider implementations live?”
- `packages/ai/src/providers/`

### “Where are sessions stored and restored?”
- `packages/coding-agent/src/core/session-manager.ts`

### “Where is the system prompt assembled?”
- `packages/coding-agent/src/core/system-prompt.ts`

### “Where are extensions loaded?”
- `packages/coding-agent/src/core/resource-loader.ts`
- `packages/coding-agent/src/core/extensions/`

### “Where does the terminal renderer live?”
- `packages/tui/src/tui.ts`

---

## Final Advice

Do not try to understand Pi by reading every file in order.

Instead:
1. trace one prompt
2. trace one tool call
3. trace one session restore
4. trace one provider stream
5. trace one extension hook

Pi is modular enough that once those five paths are clear, the rest of the codebase becomes much easier to navigate.
