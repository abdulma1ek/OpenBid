# Session logs

One file per work session, written at the end by `/end-session` (or by hand). Never edit another person's log. Logs are the record of progress between Sunday reviews.

**File name:** `YYYY-MM-DD-<name>-<topic>.md`, for example `2026-10-06-sara-signin.md`.
Sunday reviews: `YYYY-MM-DD-review-week-N.md`.

**Template** (keep it under 25 lines):

```markdown
# 2026-10-06 · <name> · <topic>

**Plan item:** W1-5
**State:** done | partly done | blocked
**Branch / pull request:** <name>/w1-signin · #12

## What changed
- …

## Documents changed
- Which plan or specification file, what changed and why. "None" if none.

## Checks
- lint, typecheck, tests: passed | failed (which, why)

## For my partner
- New migration, environment variable, dependency, or anything in your lane. "Nothing" if none.

## Next step
- …

## Plan changes to raise on Sunday
- "None", or what should move and why.
```
