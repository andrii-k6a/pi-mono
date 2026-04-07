# Open Questions

## Open

- What is the precise responsibility boundary between `AgentSession`, `Agent`, and `agent-loop`?
- How does `SessionManager` represent the current branch in JSONL?
- Where are non-message state changes such as model changes and thinking-level changes persisted?
- In concrete provider code, where are raw provider stream events translated into Pi's normalized assistant event stream?
- What is the exact ordering between retries, compaction checks, and final session persistence?

## Answered

- None yet.

## Disproven

- None yet.
