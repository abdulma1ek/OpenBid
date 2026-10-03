# OpenBid

**Repository:** https://github.com/abdulma1ek/OpenBid (private; default branch `main`). OpenBid is the working product name.

Paid B2B web app for Canadian facilities-and-goods suppliers bidding on government tenders.
Headline feature: check a tender package against the company's saved evidence, with a page and quote for every requirement. Supporting features: tender radar, bid calendar, alerts.

Built by two founders, mostly AI-assisted. Optimise for few moving parts, small testable modules, and tests that catch what a human reviewer would miss.

## Read before working

| File | Read when |
|---|---|
| `docs/architecture.md` | Any feature work. Source of truth for components, pipelines and rules. |
| `docs/data-model.md` | Touching the database, RLS, or any query. |
| `docs/decisions.md` | Before proposing a new service, library, or a change to an agreed choice. |
| `docs/team/coordination.md` | Every session. Lanes, hot files, branch and agent rules for two founders working in parallel. |
| `docs/plan/week-NN.md` (current week) | Every session. The task list and owners. `docs/plan/execution-plan.md` is the overall plan. |
| `docs/logs/` (most recent) | Every session. What changed since you last worked. |

If code and these docs disagree, stop and say so. Do not silently follow either.

## Stack (fixed; changing it requires a new entry in `docs/decisions.md`)

- TypeScript (strict), Node LTS, pnpm, one repository, no monorepo tooling.
- Next.js App Router on Vercel, functions region `yul1`.
- Supabase project in `ca-central-1`: Postgres, Auth, Storage. Plain SQL migrations in `supabase/migrations/`.
- Inngest for all scheduled and background work.
- Claude API through `@anthropic-ai/sdk`: `claude-opus-5-5` (extraction), `claude-haiku-4-5` (triage, email parsing, certificate reading).
- Postmark (outbound + inbound), Stripe (Checkout, Billing, Customer Portal), Sentry, PostHog.
- Tailwind + shadcn/ui, Zod, Vitest, Playwright.

## Hard rules

1. **Model extracts, code decides.** No model output may set a requirement result (`met`, `gap`). Results come only from `lib/assessment/` rules.
2. **No quote, no claim.** A requirement is `quote_verified = true` only if `lib/checker/verify-quote.ts` finds the quote on the stated page.
3. **Unknown is the default.** Missing data, parse failure, model refusal, timeout: result is `unknown` or the check is `failed`. Never a pass.
4. **Tenant isolation.** Every org-owned table has `organization_id`, RLS enabled, policies, and a test in `tests/rls/`. A migration adding a table without all four is incomplete.
5. **Service-role key is server-only** and used only in `lib/db/admin.ts` (jobs, webhooks). User requests use the user's session so RLS applies.
6. **All model calls go through `lib/llm/`.** No direct SDK use elsewhere. Every response is validated with Zod and checked for `stop_reason`.
7. **No customer document content** in logs, Sentry, PostHog, Inngest event payloads, or Inngest step return values. Pass IDs.
8. **Wording.** Never write "eligible", "compliant", "qualified", or "you will win" in UI, emails or marketing. Use "met by evidence", "gap", "unknown".
9. **Official source wins.** Every tender shows its source link. Every deadline message includes "verify on the official notice before submitting".
10. **Collectors** read public listing pages only: no logins, no CAPTCHA handling, no document downloads, rate-limited, with a kill switch. See `docs/architecture.md` section 5.
11. **Secrets** never in Git, chat, or client bundles. Only `NEXT_PUBLIC_*` values reach the browser.
12. **Migrations are append-only.** Never edit an applied migration; add a new one.

## Session protocol

**Start.** `/start-session <plan item>`: pull `main`, read the week file and recent logs, state the goal.

**During.** One agent, one branch, one plan item. Stay in the owner's lane (`docs/team/coordination.md` section 4). Before touching a hot file or the other founder's lane, stop and say so.

**Wrap-up.** When the user says the session or the day is over ("we're finishing up", "wrapping up", "that's it for today", "done for now"), or runs `/end-session`, do all of this before your final reply, without being asked:

1. Run lint, typecheck and tests if code exists; record the real results.
2. Apply the update rules below. If a founder decided a change this session that the plan or the specification states, update that document now with a change note. Fix or list anything missing.
3. Write the session log to `docs/logs/YYYY-MM-DD-<name>-<topic>.md` using the template in `docs/logs/README.md`: plan item, what changed, documents changed and why, check results, what the partner needs to know, next step, plan changes to raise on Sunday.
4. Commit the work and the log on the current branch and push. If the current branch is `main`, ask before pushing.
5. Reply with the state (done, partly done, blocked), the log's path, and the next step.

A session without a log is not finished. If the user ends abruptly, write the log before anything else.

## Update rules

Documentation that no longer matches the code is a bug. Make these updates in the same change, never later.

| If you change | Also update |
|---|---|
| Tables, columns, status values, policies | `docs/data-model.md` |
| A library, an external service, or an agreed choice | `docs/decisions.md` (new entry; never rewrite old ones) |
| Components, pipelines, jobs, limits | `docs/architecture.md` |
| Environment variables | `.env.example`, and tell the partner in the log |
| Folder layout or working rules | `docs/team/coordination.md` and this file, by pull request both founders approve |
| Scope, order or dates of work | `docs/plan/`, with a change note, when a founder decides it in the session. Your own suggestions go in the log under "Plan changes to raise on Sunday" |

Never edit `docs/plan/`, the specification, or this file on your own initiative or as a side effect of feature work. A founder's decision is required.

## Change notes

When a statement in the plan or the specification stops being true, never delete it or rewrite it silently. Keep the old line, struck through, and put a dated note directly beneath it:

```
~~Reminders default to 14 days, 7 days, 72 hours, 24 hours.~~
> **Changed 2026-10-07:** reminders default to 7 days and 24 hours. Why: design partners found four emails too many. Log: `2026-10-07-sara-reminders.md`.
```

- The note says what is true now, why it changed, and which session log has the detail.
- In a table, strike through the old row and add the new row directly beneath it, starting with `Changed YYYY-MM-DD:`.
- Applies to `docs/architecture.md`, `docs/data-model.md`, `docs/plan/*` and `docs/team/coordination.md`. `docs/decisions.md` gets a new entry naming the one it supersedes. This file records its changes in the list at the bottom, so the rules stay short.
- Read struck-through lines as history, not as instructions.
- New material that contradicts nothing needs no note; mention it in the log.

## Working conventions

- One feature per branch, small pull requests, tests in the same PR.
- Commits carry the human author's name only. No AI co-author or "generated with" lines in commits or pull requests.
- New dependency or external service: add an entry to `docs/decisions.md` first.
- A change to tables or statuses: update `docs/data-model.md` in the same PR.
- Money in integer cents with a currency code. Timestamps `timestamptz` in UTC; display in the user's zone and name the zone on deadlines.
- English UI strings live in one place (`lib/copy/`) so French can be added later.

## Commands

```
pnpm dev            # local app
pnpm test           # unit + RLS tests
pnpm test:e2e       # Playwright
pnpm lint && pnpm typecheck
pnpm db:migrate     # apply migrations to local Supabase
pnpm db:types       # regenerate TypeScript types from the schema
```

(These scripts are the target set; create them during the Foundation stage.)

## Changes to these rules

- 2026-10-04: wrap-up may update the plan and the specification, using change notes. Before: the plan changed only at the Sunday review. Why: full transparency between sessions, so the next session sees what changed and why.
