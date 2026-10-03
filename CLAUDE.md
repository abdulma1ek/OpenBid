# BidScope One

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

- Start with `/start-session <plan item>`; end with `/end-session`. Every session ends with a log in `docs/logs/`.
- One agent, one branch, one plan item. Stay in the owner's lane (`docs/team/coordination.md` section 4).
- Never edit `docs/plan/`, the specification, or this file as a side effect of feature work. Put proposals in the session log.
- Before touching a hot file or the other founder's lane, stop and say so.

## Working conventions

- One feature per branch, small pull requests, tests in the same PR.
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
