# Knowledge Base

This directory is the persistent study state for learning Pi internals in this repository.

## Operating Principle

Keep one small canonical state file and a few supporting artifacts.

- `state.md` is the current truth.
- `sessions/` is history.
- `artifacts/` is for deeper traces and durable notes.
- `concepts.md` and `source-map.md` are compressed knowledge.

## Canonical Files

- `state.md` — current progress, next step, and resume protocol
- `source-map.md` — curated map of important code paths and files
- `concepts.md` — validated mental models and distinctions
- `open-questions.md` — unresolved or partially resolved questions

## Session Logs

- `sessions/YYYY-MM-DD.md` — append-only session closeout notes

## Artifacts

Use `artifacts/` for targeted deep dives such as:
- prompt flow trace
- session format understanding
- provider abstraction notes
- TUI rendering notes

## Workflow

### Resume
Run `/study`.

That workflow should:
1. check the branch
2. restore state from `state.md`
3. read only currently relevant files
4. summarize current understanding
5. continue as an interactive guide

### Close out
Run `/study-close`.

That workflow should:
1. update `state.md`
2. append to today's session log
3. keep only durable knowledge in long-lived files

## Branch Convention

Preferred branch for this workflow:
- `kb`
- or `kb/<topic>`

The workflow should check the branch and ask before continuing if the branch does not match.
