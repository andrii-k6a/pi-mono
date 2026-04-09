# Open Questions

## Open

- What is the precise responsibility boundary between `AgentSession`, `Agent`, and `agent-loop`?
- How does `SessionManager` represent the current branch in JSONL?
- In concrete provider code, where are raw provider stream events translated into Pi's normalized assistant event stream?
- What is the exact ordering between retries, compaction checks, and final session persistence?
- How does `main.ts` route into each run mode concretely?

## Answered

- Non-message session state changes are persisted as dedicated session entry types, not only as message entries.
- Skills are passed to LLMs as prompt text: metadata in the system prompt, and full skill content either via model-initiated `read` of `SKILL.md` or explicit `/skill:name` expansion into a user message.
- Skill metadata XML is appended near the end of the system prompt; explicit `/skill:name` expansion places the full skill block before trailing user text in one user message.
- `AGENTS.md`/`CLAUDE.md` loading walks from `cwd` to filesystem root, preferring `AGENTS.md` per directory, with global agent-dir context first and final ordering from outermost to innermost.
- Pi's “no background bash” philosophy rejects a built-in background-job subsystem, but Pi can still invoke tmux commands and delegate long-lived terminal workflows to tmux.

## Disproven

- Skills are not exposed to providers through a special API contract or tool schema.
- “No background bash” does not mean Pi must never create tmux panes/windows; it means Pi core does not standardize that into a built-in managed jobs feature.
