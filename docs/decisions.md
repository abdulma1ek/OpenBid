# OpenBid: Decision Log

Append new entries at the bottom. Do not rewrite old ones; supersede them with a new entry that names the old ID.

Format: **ID. Decision.** Why. Rejected alternatives. Revisit when.

## Product

**D1. Paid only, with a 14-day trial.** Every document check costs real AI money, and the founders want a paid service. Rejected: permanent free tier (ongoing cost for non-payers). Revisit: if trial-to-paid conversion is poor.

**D2. First segment is facilities and goods suppliers.** Easier to approach, high tender volume, repeatable requirements (insurance, WSIB, bonding). Consequence: most of their buyers are municipal, so non-federal intake is first-class. Revisit: after 20 customer interviews.

**D3. The Eligibility Checker is the headline feature.** Tender alerts are sold at $0–$50 by several Canadian tools (TenderBridge, Wonable, GovBid, BidFit). None observed tie each requirement to a page and quote and test it against stored company evidence. Suppliers on MERX report paying for documents and then failing a mandatory criterion. Rejected: Radar + Calendar first.

**D4. Evidence is uploaded once and reused.** Certificates are read by AI, confirmed by the user, stored on the company profile, and applied to every check. Rejected: form-only entry (more onboarding effort); per-check upload.

**D5. Never claim eligibility.** Results are `met`, `gap`, `unknown`. Legal and trust reasons.

**D6. English UI; reads French documents.** French UI later; strings centralised now.

**D7. Teams with roles from the first release.** Retrofitting sharing is expensive.

**D8. Market Intelligence in Release 2; Partner Network parked; bid drafting reserved.** `files.kind = 'past_bid'` exists so drafting can be added without a schema change.

**D25. The working name is OpenBid.** Decided 2026-10-04. The brainstorm guide in `docs/reference/` uses the earlier name BidScope One.

## Coverage

**D9. Release 1 intake: CanadaBuys open data, uploads, forwarded alert emails.** Open data is federal only (the dataset documentation says provincial and municipal notices are excluded). Upload and email forwarding cover every other portal using the customer's own access.

**D10. Municipal collectors are designed in but gated.** MERX and bids&tenders public listings, metadata only, in Release 2. Each must pass: terms of use allow it, one week of stable runs, kill switch and alarm tested. Drop any that fail. Rejected: scraping behind logins; licensing ProcureData at launch (does not list Ontario's portal; resale terms unpublished).

**D11. Ontario Tenders Portal and Biddingo are not collected.** robots.txt disallows the Ontario application and it needs a login; Biddingo blocks automated agents. Covered by D9.

## Technology

**D12. TypeScript, one repository, Next.js on Vercel (`yul1`).** One language for two AI-assisted builders. Same stack class as every small competitor observed, so tooling and examples are abundant.

**D13. Supabase (`ca-central-1`) for Postgres, Auth and Storage.** One vendor, data at rest in Canada, RLS for tenant isolation. Rejected: Cloudflare R2 (extra vendor, no benefit at this size); Clerk (extra vendor).

**D14. Inngest for background work.** Retries, schedules and run history with no servers; job code runs inside our Vercel functions. Rejected: GitHub Actions cron (no retries or visibility); Supabase pg_cron alone (skipped runs are not retried, no alerting); Trigger.dev (runs our code on their infrastructure). Constraint: payloads and step outputs carry IDs only.

**D15. Claude API directly; Opus 5.5 for extraction, Haiku 4.5 for triage, email parsing and certificate reading.** Extraction accuracy is the product. All calls go through `lib/llm/` so the provider can be swapped.

**D16. US AI processing is accepted and disclosed.** No Canadian inference option exists on the Claude API, and Bedrock's Canada region does not run Claude in-region. Consequence: Protected B customers are out of scope. Revisit: when a Canadian inference option appears or an enterprise customer requires it.

**D17. Quotes are verified by our own code.** Claude's citations cannot be combined with structured output, so the model returns page and quote as fields and `verify-quote.ts` confirms them against stored page text.

**D18. Results come from deterministic rules.** The model never sets `met` or `gap`.

**D19. Postgres full-text search; no embeddings in Release 1.** Volume is a few thousand open tenders. Add pgvector only if measured recall is poor.

**D20. Postmark for email, including inbound.** Separate streams suit CASL; inbound parsing powers D9. Inbound requires the Pro plan ($16.50/month).

**D21. Stripe Checkout and Customer Portal.** No card data in our systems.

**D22. Plain SQL migrations and generated types; no ORM.** Queries run through the Supabase client under the user's session so RLS always applies.

## Budget

**D23. Under $100/month while building; $100–300 acceptable after launch.** Free tiers during the build; Vercel Pro and Supabase Pro before the first paying customer (Vercel's free plan is non-commercial; Supabase free projects pause and have no backups).

**D24. AI spend is capped per plan and globally.** Estimated $1–2 per large package; to be measured on ten real packages before plan limits are set.
