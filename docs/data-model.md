# BidScope One: Data Model

Version 1.0, 2026-10-04. Postgres on Supabase. Companion to `architecture.md`.

Conventions: `id uuid primary key default gen_random_uuid()`; `created_at` / `updated_at timestamptz` on every table; money as `*_cents bigint` + `currency text`; enums as `text` with a `check` constraint; **Org** column = table has `organization_id` and RLS.

## Accounts

| Table | Org | Key columns | Notes |
|---|---|---|---|
| `organizations` | self | `legal_name`, `trade_name`, `time_zone`, `inbound_email_token` | Token forms the unique forwarding address |
| `memberships` | yes | `user_id`, `role` (`owner`,`admin`,`member`) | Unique `(organization_id, user_id)`; at least one owner |
| `invitations` | yes | `email`, `role`, `token_hash`, `expires_at`, `accepted_at` | |
| `subscriptions` | yes | `stripe_customer_id`, `stripe_subscription_id`, `plan`, `status`, `trial_ends_at`, `current_period_end` | Written only by Stripe webhook job |

## Company

| Table | Org | Key columns | Notes |
|---|---|---|---|
| `company_profiles` | yes (PK) | `summary`, `keywords text[]`, `unspsc_segments text[]`, `regions text[]`, `min_value_cents`, `max_value_cents`, `languages text[]` | One per organization |
| `files` | yes | `storage_key`, `kind` (`evidence`,`tender_document`,`past_bid`), `mime_type`, `size_bytes`, `sha256`, `page_count`, `deleted_at` | `past_bid` reserved for future drafting |
| `evidence_items` | yes | `type`, `issuer`, `reference_number`, `amount_cents`, `currency`, `valid_from`, `expires_on`, `details jsonb`, `file_id`, `state` (`extracted`,`confirmed`,`rejected`,`expired`), `confirmed_by`, `confirmed_at` | Only `confirmed` and unexpired items are used in assessment |

`evidence_items.type`: `liability_insurance`, `auto_insurance`, `wsib`, `bonding`, `licence`, `certification`, `security_clearance`, `other`.

## Tenders

| Table | Org | Key columns | Notes |
|---|---|---|---|
| `sources` | no | `id text` (`canadabuys`,`upload`,`email`,`merx`,`bidsandtenders`), `enabled`, `last_run_at` | `enabled` is the kill switch |
| `tenders` | `owner_org_id` nullable | `source_id`, `external_id`, `title`, `title_fr`, `buyer`, `description`, `description_fr`, `unspsc text[]`, `category`, `regions text[]`, `notice_type`, `selection_criteria`, `published_at`, `closes_at`, `status` (`open`,`closed`,`cancelled`,`awarded`), `source_url`, `content_hash`, `search tsvector` | Unique `(source_id, external_id, owner_org_id)`. RLS: readable when `owner_org_id is null` or member |
| `tender_versions` | inherits | `tender_id`, `version_no`, `content_hash`, `raw jsonb`, `diff jsonb`, `observed_at` | One row per detected change |
| `tender_attachments` | inherits | `tender_id`, `url`, `title`, `language` | Links from the source; not downloaded automatically |

## Checks

| Table | Org | Key columns | Notes |
|---|---|---|---|
| `checks` | yes | `tender_id`, `state` (`queued`,`processing`,`needs_review`,`reviewed`,`failed`), `failure_reason`, `model`, `prompt_version`, `cost_cents`, `created_by` | |
| `check_documents` | yes | `check_id`, `file_id`, `ordinal` | |
| `document_pages` | yes | `file_id`, `page_no`, `text`, `is_scanned` | Source for quote verification |
| `requirements` | yes | `check_id`, `file_id`, `type`, `text`, `mandatory` (`yes`,`no`,`uncertain`), `page_no`, `quote`, `quote_state` (`verified`,`not_found`,`unverifiable_scan`), `threshold jsonb`, `due_at`, `result` (`met`,`gap`,`unknown`,`not_applicable`), `result_rule`, `evidence_item_id`, `review_state` (`pending`,`confirmed`,`corrected`,`dismissed`), `reviewed_by`, `reviewed_at`, `original jsonb` | `original` keeps the model's version when a user corrects it |

## Pursuit

| Table | Org | Key columns | Notes |
|---|---|---|---|
| `matches` | yes | `tender_id`, `score int 0..100`, `factors jsonb`, `blockers jsonb`, `state` (`new`,`saved`,`dismissed`), `dismiss_reason` | PK `(organization_id, tender_id)` |
| `bids` | yes | `tender_id`, `stage` (`deciding`,`preparing`,`submitted`,`won`,`lost`,`withdrawn`), `owner_id`, `decision`, `notes` | |
| `tasks` | yes | `bid_id`, `title`, `due_at`, `locked_to_source bool`, `assignee_id`, `state` (`open`,`done`), `requirement_id` | |
| `notifications` | yes | `user_id`, `type`, `payload jsonb`, `channel` (`in_app`,`email`), `sent_at`, `delivery_state` | Payload holds IDs, not document text |
| `notification_preferences` | yes | `user_id`, `reminder_offsets int[]`, `digest_enabled`, `digest_hour` | |

## Operations

| Table | Org | Key columns | Notes |
|---|---|---|---|
| `source_runs` | no | `source_id`, `started_at`, `finished_at`, `rows_seen`, `rows_changed`, `error` | Drives zero-row alarm |
| `inbound_emails` | yes | `from_address`, `subject`, `received_at`, `parse_state`, `tenders_created int`, `raw_storage_key` | |
| `usage_events` | yes | `feature`, `model`, `input_tokens`, `output_tokens`, `cost_cents`, `check_id` | Basis for allowances and budget cap |
| `audit_events` | yes | `user_id`, `action`, `object_type`, `object_id`, `ip` | Append-only |

## Release 2 (designed later)

`awards`, `contracts` (federal award notices and proactive-disclosure contracts over $10,000), `buyers`, `suppliers` with name-normalization and a link-confidence field.

## RLS pattern

```sql
create function is_member(org uuid) returns boolean
language sql stable security definer as $$
  select exists (select 1 from memberships m
                 where m.organization_id = org and m.user_id = auth.uid());
$$;

alter table checks enable row level security;
create policy checks_rw on checks
  for all using (is_member(organization_id)) with check (is_member(organization_id));
```

Every org-owned table follows this pattern, with role checks added where writes are restricted. Each has a test in `tests/rls/` using two organizations.

## Indexes to create with the tables

`tenders (status, closes_at)`, `tenders using gin (search)`, `tenders using gin (unspsc)`, `tenders using gin (regions)`, `matches (organization_id, state, score desc)`, `requirements (check_id)`, `tasks (organization_id, due_at) where state = 'open'`, `evidence_items (organization_id, expires_on)`, `usage_events (organization_id, created_at)`.
