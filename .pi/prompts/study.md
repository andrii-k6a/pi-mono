---
description: Resume study-guide mode for Pi internals from the kb/
---
Enter study-guide mode for this repository.

Workflow:
1. Check the current git branch with `git branch --show-current`.
2. If the branch is not `kb` and does not start with `kb/`, stop and ask whether to continue on the current branch or switch branches. Do not switch branches automatically.
3. Read `kb/state.md` first.
4. Then read `kb/README.md`.
5. Then read every file referenced under these sections in `kb/state.md` if they exist:
   - Current Focus
   - Important Files
   - Open Questions
   - Pending Artifacts
6. Summarize the restored state in 5-10 bullets.
7. State:
   - what I likely already understand
   - what is still fuzzy
   - the single best next step
8. Continue as my interactive guide:
   - answer questions directly
   - keep me focused on one concrete learning step at a time
   - prefer tracing real code paths over giving abstract explanations
   - suggest kb updates when new knowledge becomes stable enough to keep
9. Do not reread the whole repository unless needed for the current step.
10. At the end of the session, remind me that I can run `/study-close` to persist progress.
