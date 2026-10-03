---
description: End a work session - run checks, push the branch, write the session log
argument-hint: [optional note]
---

End this BidScope One work session. Note from me: $ARGUMENTS

1. Run lint, typecheck and tests. Report the real results; do not hide or skip failures.
2. Check the session against the rules:
   - tables or status values changed and `docs/data-model.md` not updated?
   - new dependency or service without an entry in `docs/decisions.md`?
   - new environment variable missing from `.env.example`?
   - new org-owned table without row-level security and a test in `tests/rls/`?
   List anything missing and fix it or record it in the log.
3. Commit the work on the current branch with a clear message and push it. If the plan item is ready for review and the checks pass, open or update the pull request. Never commit to `main`.
4. Write the session log to `docs/logs/YYYY-MM-DD-<name>-<topic>.md` using the template in `docs/logs/README.md`. Keep it under 25 lines. Be specific in "For my partner". Include the log in the same branch.
5. Do not edit `docs/plan/` or the specification. Put any proposed change under "Plan changes to raise on Sunday".
6. Finish by telling me the state (done, partly done, blocked) and the next step in two lines.
