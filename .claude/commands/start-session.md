---
description: Start a work session - sync, read the plan and recent logs, agree the goal
argument-hint: [plan item, e.g. W2-3]
---

Start an OpenBid work session. Plan item requested: $ARGUMENTS

1. Run `git status`. If there are uncommitted changes, stop and ask what to do with them. Otherwise switch to `main` and pull.
2. Read `docs/plan/execution-plan.md` (current week only), the current `docs/plan/week-NN.md`, and the five most recent files in `docs/logs/`.
3. Read sections 4, 5 and 7 of `docs/team/coordination.md`.
4. Tell me, in under ten lines:
   - where the week stands against its Sunday deliverable;
   - anything my partner changed since my last log that affects me (migrations, environment variables, dependencies, files in my lane);
   - the plan item for this session. If none was given, suggest the next unowned-by-partner item that belongs to me.
5. Read the parts of `docs/architecture.md` and `docs/data-model.md` that the item needs.
6. Create or switch to the branch `<name>/w<week>-<short-topic>`, taking `<name>` from the first word of `git config user.name`, lowercased.
7. State the session goal, the folders you expect to touch, and what "done" looks like. Flag any hot file or other-lane file you would need. Then wait for my go-ahead before writing code.
