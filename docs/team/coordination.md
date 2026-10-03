# OpenBid: How We Work

Version 1, 2026-10-04. For two founders building with AI assistance. Read by people and by AI agents; `/start-session` loads the relevant parts.

## 1. Principles

1. One person and one agent per piece of work.
2. Small changes, merged often. `main` always works.
3. The repository is the single place for code, specification, plan and logs.
4. Nobody edits the plan or the specification silently, human or AI.
5. Every session ends with a log.

## 2. Where things live

| Thing | Place | Who edits |
|---|---|---|
| Code | GitHub repository (private) | Through pull requests |
| Specification | `docs/architecture.md`, `docs/data-model.md`, `docs/decisions.md` | Pull request, both agree |
| AI rules | `CLAUDE.md` | Pull request, both agree |
| ~~Overall plan~~ | ~~`docs/plan/execution-plan.md`~~ | ~~Sunday review only~~ |
| Overall plan | `docs/plan/execution-plan.md` | Changed 2026-10-04: Sunday review, and at session wrap-up when a founder decided a change, with a change note (section 11). Why: transparency between sessions. |
| ~~This week's tasks~~ | ~~`docs/plan/week-NN.md`~~ | ~~Written at Sunday review; not edited mid-week~~ |
| This week's tasks | `docs/plan/week-NN.md` | Changed 2026-10-04: written at Sunday review; mid-week changes a founder decides are recorded with a change note. Why: same. |
| Session logs | `docs/logs/` | One new file per session, by its author |
| How we work | `docs/team/coordination.md` | Sunday review |
| Interview notes | `docs/research/` | Whoever ran the interview; no contact details |
| Bugs and ideas | GitHub Issues | Anyone |
| Secrets and keys | Shared password vault | Never in the repository or an AI chat |
| Supplier contact list | A spreadsheet outside the repository | Both |
| Test tender packages | `tests/fixtures/` | Public tenders only; never a customer's private file |

## 3. Repository structure

```
OpenBid/
├── CLAUDE.md                     rules every AI session loads
├── README.md
├── .claude/commands/             /start-session, /end-session
├── .github/workflows/            automated checks
├── docs/
│   ├── architecture.md           specification
│   ├── data-model.md
│   ├── decisions.md
│   ├── architecture-overview.html
│   ├── plan/
│   │   ├── execution-plan.md     the provisional plan, edited on Sundays
│   │   └── week-01.md …          one file per week
│   ├── team/
│   │   └── coordination.md       this file
│   ├── logs/
│   │   ├── README.md             template and naming
│   │   └── 2026-10-05-<name>-<topic>.md
│   ├── research/                 interview notes
│   └── reference/                original brainstorm guide (superseded by the specification)
├── app/                          screens and API routes
├── components/
├── lib/                          db, llm, sources, checker, assessment, matching, evidence, billing, notifications, copy
├── inngest/                      background jobs
├── supabase/migrations/
└── tests/                        rls, checker, assessment, matching, sources, e2e, fixtures
```

## 4. Lanes

Each plan item has one owner. Lanes say which folders that owner may change without asking.

| Lane | Folders | Default |
|---|---|---|
| Data and plumbing | `supabase/migrations/`, `lib/db/`, `lib/sources/`, `lib/llm/`, `lib/billing/`, `inngest/`, `app/api/` | Track A |
| Product surface | `app/(app)/`, `app/(auth)/`, `components/`, `lib/copy/`, `lib/notifications/`, `lib/evidence/` | Track B |
| Checker | `lib/checker/`, `lib/assessment/`, check screens | Split per item in the week file |
| Tests | `tests/<area>/` | Whoever owns the code under test |

Needing a file in the other lane is normal. Message your partner first, then change it in its own small pull request.

## 5. Hot files

These cause most collisions. Each has one rule.

| File | Rule |
|---|---|
| `supabase/migrations/*` | One new timestamped file per change. Never edit a merged one. Tell your partner when one merges. |
| `lib/db/types.ts` | Generated. Never edit by hand. On conflict, take `main` and run `pnpm db:types`. |
| `package.json`, lockfile | Add a dependency in its own pull request and merge it the same day. On conflict, take `main` and reinstall. |
| `.env.example` | Update in the same pull request that needs the variable; put the value in the vault; tell your partner. |
| `CLAUDE.md`, specification files | Change only by pull request that both approve. |
| ~~`docs/plan/*`~~ | ~~Sunday review only. Mid-week proposals go in your log.~~ |
| `docs/plan/*` | Changed 2026-10-04: a founder's decision is recorded at wrap-up with a change note; an AI's suggestion still goes in the log. Tell your partner, since you may both touch this file. |
| `docs/logs/*` | Add your own file. Never edit someone else's. |

## 6. Branches and pull requests

- Never push to `main`. It is protected: pull request, one approval, checks passing.
- Branch name: `<name>/w<week>-<short-topic>`, for example `sara/w2-invitations`.
- One plan item per branch. Keep a branch alive two days at most.
- Aim for pull requests the other person can read in ten minutes.
- The other founder reviews. Running `/code-review` on the pull request is a good first pass, not a substitute for looking.
- Squash-merge, delete the branch, and pull `main` before starting anything new.
- No force-pushing a branch your partner has checked out.

## 7. Working with AI agents

**Session routine**

| Moment | Do |
|---|---|
| Start | Run `/start-session <item>`. It pulls `main`, reads the week file and recent logs, and states the goal. |
| During | One agent, one branch, one plan item. Stay in your lane's folders. |
| End | Say "we're finishing up" or run `/end-session`. Either way the agent runs the checks, summarises the session, writes the log to `docs/logs/`, and pushes. A session without a log is not finished. |

**Rules for the agent**

1. Never run two agents on the same branch or in the same folder. For parallel work, use a separate worktree per agent.
2. If the work needs a hot file or the other lane, stop and say so before changing it.
3. Never change the plan, the specification or `CLAUDE.md` as a side effect. Put the proposal in the log.
4. A new library or service needs an entry in `docs/decisions.md` first.
5. Run the checks before every pull request and report failures plainly.
6. If the specification is unclear or contradicts the code, stop and ask.
7. Keep the documents true: follow the update rules in `CLAUDE.md` in the same change.
8. No secrets and no customer documents in prompts. Use the public packages in `tests/fixtures/`.
9. When a founder decides a change during a session, record it at wrap-up in the document it affects, using a change note (section 11).

**What to give an AI outside Claude Code** (a chat tool, or a fresh assistant with no repository access)

| Task | Upload |
|---|---|
| Any session | `CLAUDE.md`, `docs/team/coordination.md`, the current `docs/plan/week-NN.md`, logs since your last session |
| Building a feature | plus `docs/architecture.md` |
| Database or permissions | plus `docs/data-model.md` |
| Proposing a new tool or approach | plus `docs/decisions.md` |
| Sunday review or replanning | `docs/plan/execution-plan.md` and the week's logs |

Inside Claude Code in this repository nothing needs uploading.

## 8. Sunday review

Thirty to forty-five minutes, together.

1. Demonstrate the week's deliverable on staging.
2. Decide: passed or not, against "passes when".
3. Read the week's logs, especially "Plan changes to raise".
4. Edit `execution-plan.md` and add a line to its change table.
5. Write next week's `week-NN.md` with an owner on every item.
6. Write a review log named `YYYY-MM-DD-review-week-N.md`.
7. Merge all of it in one pull request.

## 9. When to message your partner

- You need a hot file or a file in their lane.
- You merged a migration, added an environment variable, or added a dependency.
- You are blocked for more than an hour.
- You think a gate will fail.

## 10. Repository settings

- Private repository; both founders are admins; two-factor sign-in required.
- `main` protected: pull request required, one approval, automated checks required.
- Vercel builds a preview link for every pull request.

## 11. Change notes

Added 2026-10-04. When a line in the plan or the specification stops being true, it is never deleted. It is struck through, and a dated note directly beneath it says what is true now, why it changed, and which session log has the detail.

```
~~Reminders default to 14 days, 7 days, 72 hours, 24 hours.~~
> **Changed 2026-10-07:** reminders default to 7 days and 24 hours. Why: design partners found four emails too many. Log: `2026-10-07-sara-reminders.md`.
```

In a table, strike through the old row and add the new row beneath it. Struck-through lines are history, not instructions. The full rule is in `CLAUDE.md`.
