# Week 1: set up and deploy (Oct 5–11)

**Sunday deliverable:** a staging site where both founders can sign in.
**Passes when:** sign-in works on the staging link, and a pull request runs automated checks.

Owners are a starting suggestion. Decide who is A and who is B before Monday and write the names here.

| ID | Item | Owner | Done when |
|---|---|---|---|
| W1-1 | Open accounts with two-factor sign-in: GitHub (private repo, both as admins), Supabase (two projects in Canada Central), Vercel, Inngest, Anthropic API (with a monthly spend limit), Sentry, shared password vault | Both | Both can sign in to each; keys are in the vault |
| W1-2 | Make this folder the repository and push it; protect `main` | A | Direct pushes to `main` are refused |
| W1-3 | Scaffold the app: Next.js, strict TypeScript, Tailwind, shadcn/ui, the scripts named in `CLAUDE.md` | A | `pnpm dev`, `pnpm lint`, `pnpm typecheck`, `pnpm test` all run |
| W1-4 | Automated checks on every pull request, including that it adds a session log in `docs/logs/` | A | A pull request shows pass or fail; one without a log fails |
| W1-5 | Sign up and sign in with Supabase Auth | A | A new user can sign in locally |
| W1-6 | App shell: layout, navigation placeholders, sign-in screens, central wording file | B | Signed-in user sees the empty dashboard |
| W1-7 | Deploy to Vercel, functions in the Montreal region, staging environment variables | B | Staging link works for both founders |
| W1-8 | A first background job through Inngest | B | The job shows as completed in Inngest for staging |
| W1-9 | Answer open items: Vercel function time limit; MERX and bids&tenders terms of use | Both | Answers recorded in `docs/decisions.md` |
| W1-10 | List 30 target suppliers; save three real public tender packages in `tests/fixtures/` | Both | List exists outside the repo; three packages are in the repo |

Postmark's paid plan is not needed until week 8.
