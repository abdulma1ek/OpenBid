# BidScope One: Execution Plan (provisional)

Version 1, written 2026-10-04. Weeks run Monday to Sunday. Every week ends with one deliverable that can be shown on Sunday.

This file is edited **only at the Sunday review** (`../team/coordination.md` section 8). Day-to-day progress lives in `../logs/`. The detailed task list for the current week lives in `week-NN.md` beside this file and is written one week at a time.

## How to read it

- **Deliverable:** something that exists and can be demonstrated on Sunday.
- **Passes when:** the check that decides whether the next week starts as planned. If it fails, the next week finishes it first and later weeks shift.
- **Gate weeks (2, 6, 9):** do not move past these until they pass. They protect tenant isolation, checker quality, and payments.
- **Tracks:** a default split so two people rarely touch the same files. Reassign at any Sunday review.

| Track | Covers |
|---|---|
| A: data and plumbing | Database, sign-in, isolation, background jobs, tender intake, billing |
| B: product surface | Screens, onboarding, evidence vault, review screen, emails, wording |
| Both | Checker pipeline (weeks 4–6), Sunday review |
| Outside the code | Supplier interviews, real tender packages, design partners, company and Stripe setup |

## Timeline

| Week | Ends Sunday | Stage | Deliverable | Passes when |
|---|---|---|---|---|
| 1 | Oct 11 | Foundation | A staging site where both founders can sign in | Sign-in works on the staging link; a pull request runs automated checks |
| 2 | Oct 18 | Foundation **(gate)** | Companies, teams, invitations and roles | Two test companies cannot see each other's data, proven by automated tests |
| 3 | Oct 25 | Evidence vault | Company profile and certificate upload with AI reading | A real insurance certificate and a WSIB clearance become confirmed records with expiry dates |
| 4 | Nov 1 | Checker | Check pipeline without the polished screen | Three real packages produce requirement lists with verified quotes; cost per package is measured |
| 5 | Nov 8 | Checker | Full check on staging: upload, results, review | One package goes from upload to Met / Gap / Unknown to reviewed, end to end |
| 6 | Nov 15 | Checker **(gate)** | Quality report on ten real packages, shown to two or three suppliers | Every invented requirement is caught by the quote check; missed mandatory items are counted and judged acceptable by both founders |
| 7 | Nov 22 | Radar | Federal tenders imported and scored against a profile | The import runs unattended for three days; a test profile sees sensible matches with reasons |
| 8 | Nov 29 | Radar and calendar | Forwarded alert emails, bids, tasks, reminders, daily digest | A forwarded portal alert appears in the radar; a reminder and a digest email arrive |
| 9 | Dec 6 | Billing **(gate)** | Trial, subscription, check allowance, policies | A test company subscribes, uses its allowance, and is stopped at the limit; privacy policy and terms are on the site |
| 10 | Dec 13 | Pilot | Three to five design partners onboarded by hand | Partners have run real checks; a launch decision is written down |

## Week by week

### Week 1 (Oct 5–11): set up and deploy
- **A:** repository, project scaffold, automated checks on pull requests, sign-in.
- **B:** app shell and navigation, staging deployment in the Montreal region, a first background job that proves jobs run.
- **Outside:** open all service accounts with two-factor sign-in; list 30 target suppliers; save three real public tender packages as test material.
- **Also:** answer two open questions from the architecture: how long a server function may run on Vercel, and whether MERX and bids&tenders terms permit reading public listings.

### Week 2 (Oct 12–18): teams and isolation (gate)
- **A:** organizations, memberships, roles, database isolation policies and their tests.
- **B:** create-company flow, invitations, team settings, role-based screens.
- **Outside:** first outreach to suppliers; book interviews.

### Week 3 (Oct 19–25): evidence vault
- **A:** private file storage, the AI gateway with usage logging and spend cap, the certificate-reading job.
- **B:** onboarding and company profile, upload and confirm screens, expiry display.
- **Outside:** interviews; ask each supplier for one tender package they bid on or skipped.

### Week 4 (Oct 26–Nov 1): checker pipeline
- **A:** upload, page text extraction, requirement extraction, splitting of large packages.
- **B:** quote verification, a plain results view, hand-labelling the first test packages.
- **Outside:** interviews continue.

### Week 5 (Nov 2–8): checker review screen
- **A:** assessment rules for insurance, WSIB, bonding, licences and clearance; check states and failure handling.
- **B:** side-by-side document and checklist screen; confirm and correct actions.
- **Outside:** start company registration and bank account so Stripe can go live by week 9.

### Week 6 (Nov 9–15): checker quality (gate)
- **Both:** run ten real packages, compare by hand, fix the largest sources of error, write the quality report.
- **Outside:** show the checker to two or three suppliers and record their reactions.
- This is the week to change direction if the checker is not good enough.

### Week 7 (Nov 16–22): federal radar
- **A:** CanadaBuys import, change detection, matching filter and scoring.
- **B:** tender list and detail screens, match reasons and blockers, save and dismiss.
- **Outside:** line up design partners.

### Week 8 (Nov 23–29): forwarded alerts, bids, calendar
- **A:** inbound email parsing into private tenders, reminder and digest jobs, amendment alerts.
- **B:** start-a-bid flow, tasks, calendar view and calendar feed, notification preferences.
- **Outside:** Stripe account set up in test mode.

### Week 9 (Nov 30–Dec 6): billing and policies (gate)
- **A:** Stripe checkout and webhooks, trial, check allowance, usage limits; move hosting to paid plans.
- **B:** billing screens, privacy policy and terms pages, account deletion and export, final wording pass.
- **Outside:** confirm design partners and dates.

### Week 10 (Dec 7–13): pilot and hardening
- **Both:** onboard partners by hand, fix what they hit, test restoring a backup, run the launch checklist.
- **Outside:** collect feedback; decide launch date and price.

## After week 10 (not scheduled)

- Feasibility test of the MERX and bids&tenders collectors against their gate.
- Market Intelligence: award and contract history.
- French interface.

## Assumptions

- Each founder has roughly 15 or more hours a week. With less, stretch every stage; do not cut the gates.
- Both founders build with AI assistance and follow `../team/coordination.md`.
- Ten weeks is a target for a pilot, not a public launch.

## Changes to this plan

| Date | Change | Why |
|---|---|---|
| 2026-10-04 | Plan created | |
