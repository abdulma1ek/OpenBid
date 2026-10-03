# BidScope One: Technical Architecture

Version 1.0, 2026-10-04. Status: agreed scope. Companion files: `../CLAUDE.md` (rules), `data-model.md` (tables), `decisions.md` (why).

Facts marked **[verified]** were checked on 2026-10-04. Items marked **[verify]** are assumptions to confirm before depending on them.

## 1. Product scope

**Customer:** Canadian suppliers of facilities services and goods (cleaning, maintenance, equipment supply) selling to federal, provincial and municipal buyers. Launch area: Ontario / Ottawa.

**Commercial model:** paid only. 14-day trial with a small check allowance, then subscription tiers with a monthly check allowance. No permanent free tier.

| Feature | Release | Summary |
|---|---|---|
| Accounts and teams | R1 | Organizations with `owner`, `admin`, `member` roles |
| Evidence vault | R1 | Upload certificates once at onboarding; AI reads; user confirms; reused for every check |
| Eligibility Checker | R1 | Upload a tender package; page-cited requirement checklist compared to evidence |
| Radar | R1 | Federal tenders + forwarded alert emails, scored against the profile with visible reasons |
| Calendar and alerts | R1 | Deadlines, tasks, reminders, amendment alerts, ICS feed |
| Billing | R1 | Stripe subscription, trial, usage limits |
| Municipal collectors | R2, gated | MERX and bids&tenders public listings |
| Market Intelligence | R2 | Awards and contract history on the same database |
| AI bid drafting | Reserved | Data model leaves room; nothing built |
| Partner Network | Parked | No design |

**Non-goals:** auto-submitting bids; holding funds; claiming a company is eligible; scraping behind logins; Protected B workloads.

## 2. System components

```
Browser
  -> Next.js app on Vercel (region yul1)
       -> Supabase (ca-central-1): Postgres + RLS, Auth, Storage
       -> Claude API (US processing)            [only path where document content leaves Canada]
       -> Stripe, Postmark
Inngest -> invokes job functions hosted in the same Next.js app
Sources: CanadaBuys CSV | uploads | Postmark inbound email | municipal listing pages (R2)
```

| Component | Service | Region / note |
|---|---|---|
| Web app, API routes, job code | Next.js App Router on Vercel | Functions pinned to `yul1` (Montreal) **[verified available]** |
| Database | Supabase Postgres | `ca-central-1` **[verified available]** |
| Auth | Supabase Auth | Email + password and magic link |
| File storage | Supabase Storage, private buckets | Same region; access by short-lived signed URL |
| Jobs and schedules | Inngest | Orchestrator is outside Canada; receives IDs only |
| LLM | Claude API | Inference geography options are `us` / `global`; no Canada option **[verified]** |
| Email | Postmark | Transactional stream, broadcast stream, inbound stream (Pro plan) **[verified]** |
| Payments | Stripe | Checkout + Customer Portal + webhooks |
| Errors / analytics | Sentry / PostHog | Scrub request bodies and document content |

**Data residency statement (for the privacy policy):** customer files and records are stored in Canada. Document pages and profile text are sent to Anthropic's API for processing in the United States and are not used to train models.

## 3. Repository layout

```
app/                    Next.js routes
  (auth)/               sign in, sign up, invite acceptance
  (app)/                dashboard, radar, tenders/[id], checks/[id], bids, calendar, company, settings, billing
  api/
    inngest/            Inngest handler
    webhooks/stripe/
    webhooks/postmark-inbound/
components/
lib/
  db/                   server.ts (user session, RLS on), admin.ts (service role, jobs only), types.ts (generated)
  llm/                  gateway.ts, prompts/, schemas/      <- only place the Claude SDK is imported
  sources/              adapter interface + canadabuys/, email/, merx/, bidsandtenders/
  checker/              pages.ts, extract.ts, verify-quote.ts
  assessment/           rules per requirement type
  matching/             filter.ts, score.ts
  evidence/
  billing/              plans.ts, usage.ts
  notifications/
  copy/                 all UI strings
inngest/                job function definitions
supabase/migrations/    SQL
tests/
  rls/  checker/  assessment/  matching/  sources/  e2e/
  fixtures/             real public tender packages and CSV samples (no customer data)
docs/                   specification at the root; plan/, team/, logs/, research/ beside it
.claude/commands/       /start-session and /end-session
```

## 4. Tenancy and security

- A user belongs to one or more organizations through `memberships`. The active organization is stored in the session.
- Org-owned tables carry `organization_id` and RLS policies using a SQL helper `is_member(organization_id)`; write policies check role where needed.
- `tenders` is shared when `owner_org_id is null` and private otherwise (uploads, forwarded emails).
- Roles: `owner` (billing, delete org, all), `admin` (members, profile, evidence), `member` (checks, bids, tasks).
- Storage keys are opaque: `org/<org_uuid>/<file_uuid>`. No names or titles in keys.
- Jobs use the service role and must filter by `organization_id` explicitly; each job has a test proving it cannot cross organizations.
- Audit events for: sign-in, role change, evidence change, file download, export, billing change, deletion.
- Deletion: an owner can delete files, checks, or the whole organization; a job removes storage objects.

## 5. Tender intake

All sources implement one interface and write the same normalized shape.

```ts
interface SourceAdapter {
  id: string;                       // 'canadabuys', 'email', 'merx', 'bidsandtenders'
  fetch(ctx): Promise<RawNotice[]>; // pull or receive
  normalize(raw): NormalizedTender; // title, buyer, category codes, regions, closes_at, source_url, attachments[]
}
```

Upsert key: `(source_id, external_id)`. A content hash decides whether a new `tender_versions` row is written; a new version on a tender that an organization saved or is bidding on creates an amendment notification with a field-level diff.

| Source | Mechanism | Visibility | Release | Notes |
|---|---|---|---|---|
| `canadabuys` | Download CSV: new-notices file every 2 h, open-notices file daily | Shared | R1 | 67 bilingual columns incl. UNSPSC, regions, trade agreements, selection criteria, attachment URLs. No value or clearance fields. Federal only. Some notices link to MERX with no attachment. **[verified]** |
| `upload` | User uploads a package and enters or confirms title, buyer, closing date | Private | R1 | Works for any portal |
| `email` | Each org gets a unique inbound address; user sets auto-forward of portal alert emails; Haiku parses listings | Private | R1 | Forwarding confirmation emails must be surfaced in-app **[verify Gmail/Outlook flow]** |
| `merx` | Read public open-solicitation listing pages | Shared | R2, gated | Listings load without login; robots.txt does not disallow them **[verified]**. Terms of use **[verify]** |
| `bidsandtenders` | Read per-municipality public listing pages | Shared | R2, gated | robots.txt allows with `Crawl-delay: 10` **[verified]**. Page structure **[verify]** |

**Excluded from collectors:** Ontario Tenders Portal (robots.txt disallows the application path; login required) and Biddingo (blocks automated agents). Covered by `email` and `upload`.

**Collector rules (R2):** listing metadata only (title, buyer, closing date, link); never download documents; honour robots.txt and crawl delay; honest user agent with a contact address; one request stream per host; per-source kill switch in `sources.enabled`; `source_runs` records row counts and a zero-row alarm fires after two empty runs.

**Collector gate:** terms of use permit it; one week of unattended runs without breakage; kill switch and alarm tested. Fail any: drop the collector.

## 6. Evidence vault

1. During onboarding the user uploads certificates: liability insurance, WSIB clearance, bonding capacity letter, licences, certifications, security clearance, and anything else relevant.
2. Job `evidence/read`: Haiku reads the file and returns structured fields (type, issuer, policy or certificate number, amount, currency, valid from, expiry, named insured).
3. The user reviews and confirms each field. Only `confirmed` evidence is used in assessment.
4. Expiry reminders at 60, 30 and 7 days.
5. Evidence is reused for every check; nothing is re-uploaded per tender.

## 7. Eligibility Checker pipeline

Job `check/run`, one Inngest function with steps; each step is idempotent on `(check_id, step)`.

| Step | Does | Output |
|---|---|---|
| 1 `store` | Validate type and size; store file; SHA-256 | `check_documents` row |
| 2 `pages` | Extract text per page (`unpdf` for PDF, `mammoth` for DOCX). Pages with no text layer are marked `scanned` | `document_pages` rows |
| 3 `extract` | Send the PDF to `claude-opus-5-5` with a strict output schema. Split packages over 600 pages or 32 MB, and split further if a step nears the function time limit **[verify limit]** | Candidate requirements: `text`, `type`, `mandatory`, `page`, `quote`, `threshold` |
| 4 `verify` | Normalize whitespace, hyphenation and quotes; search for `quote` on `page` (then page ±1). Scanned pages cannot be verified by text and are marked `unverifiable_scan` | `quote_verified` true/false |
| 5 `assess` | Run the rule for the requirement type against confirmed evidence | `result`: `met`, `gap`, `unknown`, `not_applicable` |
| 6 `review` | User confirms or corrects each mandatory item; corrections are stored for measuring extraction quality | `reviewed_by`, `reviewed_at` |

**Why quotes are verified in code:** Claude's citations feature cannot be combined with structured output (the API returns 400), so the model returns page and quote as ordinary fields and our code proves them.

**Requirement types (v1):** `insurance`, `wsib`, `bonding`, `bid_security`, `licence_certification`, `security_clearance`, `experience_references`, `site_visit`, `submission_format`, `deadline`, `mandatory_form`, `product_specification`, `delivery_terms`, `pricing_form`, `language`, `other`.

**Rule-testable in v1** (can produce `met` or `gap`): `insurance`, `wsib`, `bonding`, `licence_certification`, `security_clearance`. Everything else yields `unknown` plus the cited text, and becomes a checklist task.

**Mandatory flag** is three-valued: `yes`, `no`, `uncertain`. `uncertain` is displayed as mandatory.

**Check states:** `queued`, `processing`, `needs_review`, `reviewed`, `failed`.

## 8. Radar and matching

1. **Filter (SQL):** open status, region overlap, category overlap (UNSPSC segments chosen in the profile), full-text match on profile keywords in English or French.
2. **Triage (Haiku):** each surviving tender plus the profile produces structured factors with a one-line reason each: capability fit, region fit, size fit (when known), known requirements seen in the notice text.
3. **Score (code):** a weighted sum of the factors, stored with the factors. Weights live in `lib/matching/score.ts`.
4. **Blockers:** a factor flagged as a likely mandatory mismatch is displayed as a red flag regardless of score.
5. User actions: save, dismiss, start bid. Dismissals are recorded with a reason.

Embeddings and pgvector are not used in R1; add only if measured recall is poor.

## 9. Calendar, tasks and notifications

- "Start bid" creates a `bids` row and default tasks (decision, questions deadline, site visit, pricing, forms, review, submission).
- Deadlines taken from the source are locked to the source; internal buffer dates are separate fields.
- Reminders default to 14 days, 7 days, 72 hours, 24 hours; configurable per user.
- Requirements with a due date or a `gap`/`unknown` result can be turned into tasks.
- Each organization has a private ICS feed URL for Outlook and Google Calendar.
- Notifications are rows first, then delivered (in-app and email); delivery status is recorded.
- Email classes: transactional, user-requested alerts (with preferences), marketing (separate stream, CASL consent and unsubscribe).

## 10. LLM gateway (`lib/llm/`)

```ts
llm.extractRequirements({ fileId, pageRange })   // claude-opus-5-5
llm.readEvidence({ fileId })                     // claude-haiku-4-5
llm.triageTender({ tender, profile })            // claude-haiku-4-5
llm.parseAlertEmail({ emailId })                 // claude-haiku-4-5
```

Rules for the gateway:

- Structured output via `output_config.format` with a JSON schema; validate again with Zod.
- Always inspect `stop_reason`. `refusal` and `max_tokens` are failures, never empty successes.
- On `claude-opus-5-5`: do not send `thinking: {type: "disabled"}`, `budget_tokens`, or forced `tool_choice` (`any` / `tool`); all return 400. Set `output_config.effort` explicitly.
- Stream long requests and collect the final message.
- Record every call in `usage_events` with model, input tokens, output tokens, cost in cents, organization, feature.
- Before a call, check the organization's allowance and the global monthly budget; refuse with a clear error when exceeded.
- Use the Batch API (50% cost) for non-urgent bulk work such as nightly triage.
- Prompts are versioned files in `lib/llm/prompts/`; the prompt version is stored on each `checks` row.
- Published prices used for estimates: Opus 5.5 $4 / $20 per million input / output tokens; Haiku 4.5 $1 / $5.

## 11. Background jobs (Inngest)

| Function | Trigger | Does |
|---|---|---|
| `source/canadabuys.new` | cron, every 2 h | Import new-notices file |
| `source/canadabuys.open` | cron, daily | Reconcile open-notices file; close missing tenders |
| `source/email.received` | Postmark inbound webhook | Parse forwarded alert into private tenders |
| `match/run` | after import; on profile change | Filter, triage, score |
| `check/run` | check created | Section 7 |
| `evidence/read` | evidence file uploaded | Section 6 |
| `notify/digest` | cron, daily per org time zone | Daily radar digest |
| `notify/reminders` | cron, hourly | Deadline and expiry reminders |
| `notify/send` | notification created | Deliver and record status |
| `billing/sync` | Stripe webhook | Update subscription and allowance |
| `cleanup/storage` | cron, daily | Remove orphaned and deleted files |

Event payloads and step return values contain IDs and counts only. Inngest free tier: 50,000 executions per month **[verified]**.

## 12. Billing and limits

- Stripe Checkout starts a subscription; `organization_id` in metadata; webhook signature verified; handlers idempotent on event ID.
- `subscriptions` holds plan, status, period end. `lib/billing/plans.ts` maps plan to monthly check allowance and seat limit.
- A check consumes one allowance unit when it reaches `needs_review`. Failed checks are not charged.
- Trial: 14 days, small check allowance, no card required at sign-up **[decision can change before launch]**.
- Global hard cap on monthly AI spend, configurable by environment variable.

## 13. Environments and cost

| Environment | Hosting | Purpose |
|---|---|---|
| Local | Supabase CLI + `pnpm dev` + Inngest dev server | Daily work |
| Staging | Vercel preview + Supabase project `bidscope-staging` | Review and test |
| Production | Vercel production + Supabase project `bidscope-production` | Customers |

| Service | Building | From first paying customer |
|---|---|---|
| Vercel | $0 (Hobby, non-commercial only) | $20 per seat (Pro) |
| Supabase | $0 (2 projects; pause after 1 week idle; no backups) | from $25 (Pro; daily backups) |
| Inngest | $0 | $0 to 50k executions |
| Postmark | $16.50 (Pro, for inbound) | $16.50 |
| Claude API | est. $30–60 | scales with checks; est. $1–2 per large package |

Target: under $100/month while building; $100–300 acceptable after launch.

## 14. Testing and quality gates

| Layer | Tool | Must cover |
|---|---|---|
| Unit | Vitest | Assessment rules, quote verification, scoring, normalizers |
| Isolation | Vitest against local Supabase | Org A cannot read or write Org B on every org-owned table and storage path |
| Pipeline | Vitest with recorded model outputs | Checker steps on fixture packages |
| End-to-end | Playwright | Sign up, onboarding, upload evidence, run check, review, start bid |
| Extraction quality | Script over `tests/fixtures/` | Missed and invented requirements against hand-labelled packages |

CI on every pull request: lint, typecheck, unit, isolation. Merges to `main` deploy to production only after staging checks pass.

**Quality bar before charging:** ten real packages hand-compared; every invented requirement is caught by the quote check; missed mandatory requirements are counted and published internally. Do not market an accuracy figure that has not been measured.

## 15. Open items to verify

1. Vercel function time limit per plan, to size extraction steps.
2. MERX and bids&tenders terms of use.
3. bids&tenders listing page structure.
4. Extraction quality on French documents.
5. Email auto-forward setup in Gmail and Outlook.
6. Real AI cost per check on ten packages.
7. Postgres French full-text configuration quality for tender language.
