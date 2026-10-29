# GrowthOpsCo CRM — external Supabase setup

**This hosting change performs no deployment, account/repository change, live SQL or Supabase setting/data change.** The external “Growthops CRM” project (ref `biglmdldgoqojvxdhiiv`) was connected in prior work; setup indicators report configuration presence, not hosted Auth/schema health. Lovable Cloud is not required. The deployment target is **Vercel**, with TanStack Start + Nitro. **Vercel Hobby is restricted to personal/non-commercial use: this business CRM needs an eligible commercial plan for production publishing.** No purchase or upgrade is authorized or performed.

## Environment (server only)

| Variable | Purpose |
| --- | --- |
| `SUPABASE_URL` | Your external project's HTTPS URL |
| `SUPABASE_PUBLISHABLE_KEY` | Its anon JWT key or publishable key |
| `SUPABASE_SERVICE_ROLE_KEY` | Its service-role/secret key; only authenticated cron uses this |
| `CRON_SECRET` | Random 32+ character cron bearer secret |

Set these in your hosting environment; never commit values or paste private keys in chat. The app uses Supabase SSR HTTP-only cookies and server-side data access, so **no `VITE_*` variables are needed**. Never prefix the service-role key with `VITE_`. Configuration detection requires URL + public key, not the privileged key. Setup status reports presence, not connectivity/schema/auth health. Missing service key or cron secret disables cleanup only. HTTPS is required for Secure cookies.

On Vercel, privately configure the four variables in **Project Settings → Environment Variables** for Production; use separate test-project credentials for trusted previews if needed. Lovable's secure connection does not automatically configure a separate Vercel deployment. Never put values in `vercel.json` or expose production secrets to untrusted builds/previews. Vercel Cron automatically adds `Authorization: Bearer <CRON_SECRET>` to its GET request when that environment variable is configured. No dispatcher-origin variable is needed. Do not change Supabase settings or reapply the migration as part of this hosting change.

## Schema: review, then apply yourself

1. Prior connection is limited to external “Growthops CRM” (`biglmdldgoqojvxdhiiv`); the user reports the migration already applied. Do not rerun it or run SQL on a portfolio/other project. The steps below are for fresh, explicitly authorized setup only.
2. Back up the selected project's schema/data and review `supabase/migrations/20261029203500_growthops_crm.sql`.
3. Apply this migration once using its SQL editor or your existing Supabase CLI migration process. It creates **14 `crm_*` tables** to avoid colliding with unrelated public tables. It uses one transaction, explicit grants, RLS, separate roles, and atomic lifecycle RPCs. No CRM sample rows are seeded.
4. If any `crm_*` names or the `crm_signup` trigger already exist, stop and compare their schema before applying. `CREATE TABLE IF NOT EXISTS` is not an upgrade strategy for arbitrary existing tables; rollback protects a failed application. Keep a migration ledger; do not rerun an already applied migration.

All ordinary reads use a verified Supabase Auth user with their public-key client and RLS. Supabase accounts without a provisioned CRM role receive no CRM access, even if they submit role metadata during signup. The signup trigger creates profiles only; it never assigns a role.

## Private accounts and roles

In your external project's Auth settings, enable email/password sign-in and disable public signup for this internal application. These settings have **not been inspected or changed** here. Create/confirm owner and coordinator accounts privately through the Supabase dashboard. Existing accounts get a blank CRM profile during migration; new accounts get one from the trigger.

Use each account's actual Auth UUID in your SQL editor:

```sql
-- Replace UUID and display name; repeat for coordinator/viewer as needed.
update public.crm_profiles set name = 'Owner Name' where id = '<auth-user-uuid>';
insert into public.crm_user_roles(user_id, role)
values ('<auth-user-uuid>', 'owner');
```

Roles: all enabled provisioned members read CRM rows; owner/coordinator create and edit; viewer cannot write. Only owner can delete ordinary non-lead records. **Nobody can directly delete a lead, change disposition columns, write audit events or self-promote through the Data API.** Profiles permit updating only one's own name. Disabling a profile revokes CRM access on the next request. Only the service-role cron RPC permanently deletes leads.

There is no public signup or password-reset screen, preserving the existing application's scope. Use private Auth administration for password recovery. Passwords and sessions are managed by Supabase Auth, not custom tables/hashes.

## Existing data cutover — not performed

This is a **code and schema migration, not a live record migration**. Before a cutover, export/back up the previous database yourself and map its old user IDs to newly provisioned Supabase Auth UUIDs. CRM record IDs remain text so existing lead/company/contact IDs can be preserved. Map `owner_id`, `non_workable_by`, `assignee_id`, and `actor_id`; retain historical audit actor IDs as text. Convert booleans, UTC timestamps and JSON audit metadata explicitly, load in dependency order, and validate all links and deletion timestamps. Never copy custom password hashes or obsolete session rows. Review legacy audits for PII before importing. Existing expired leads must be reviewed before enabling cron. No import script runs automatically.

## Daily cleanup

`vercel.json` declares exactly one cron: `/api/public/cron/purge-non-workable`, expression `0 3 * * *` (UTC). Vercel invokes the existing **GET** handler with `Authorization: Bearer <CRON_SECRET>`. Authorization is checked before dynamically loading the server-only service-role client and calling `crm_cleanup_non_workable`. GET and POST remain supported; POST is available for authorized manual retries. Machine requests return 503 for missing configuration, 401 for bad authorization and 500 for failed cleanup; successful responses contain counts only. Do not enable a second scheduler.

Cron jobs execute only against **production deployments**, not preview deployments or local development. Hobby allows only daily cron and has hourly precision: a `0 3 * * *` job can invoke any time from **03:00 to 03:59 UTC**, not exactly 03:00. Pro/Enterprise offer per-minute precision, not an exact execution/uptime guarantee. Hobby is ineligible for this commercial CRM regardless of this schedule's technical compatibility. Cron consumes Vercel Functions usage and inherits plan/function-duration limits; inspect the generated function configuration and chosen plan before production. This change does not impose or promise a specific paid-plan duration.

Vercel does not automatically retry failed cron invocations. A function timeout can still leave an in-flight database transaction completing; inspect queue/audit before an authorized retry. Cleanup is idempotent. Unavailability, timeouts or failed jobs delay physical deletion until a successful invocation. Database deadlines and undo cutoff remain authoritative; there is no automatic backfill guarantee.

Marking uses database time truncated to milliseconds; deadline = marked time + **168 hours (604800000 ms)**, independent of daylight saving. Undo is refused at or after the deadline, including manual-review rows. The queue retains overdue rows until physical cleanup so pending cleanup/review remains visible. Physical deletion runs at the next successful daily job, not necessarily at the exact deadline.

Cleanup is one PostgreSQL transaction. Linked tables are locked against writes, due leads are row-locked, blockers are rechecked inside the transaction, and audit events commit with mutations. Active/won deals, active projects, invoices or payments flag manual review. Only the lead plus tasks with no deal/project link and activities with no deal/contact link are deleted. Shared records survive and their deleted-lead foreign key is set to NULL; companies/contacts are never deleted. Audit contains IDs, timestamps, reason/status codes and blocker counts only. Repeated/concurrent cleanup cannot duplicate the deletion audit.

## Verification and hosting

Company/contact/service screens require no new schema or remote setup changes. Existing
owner/coordinator accounts can create/edit and import; viewer is read-only but can export
readable rows. CSV templates are available inside each module; contacts use existing company
record IDs. Imports create new IDs in one INSERT, never upsert or partially save valid rows.
Invalid CSV rows or inaccessible company links block the whole import. No delete controls
were added, and lead/audit mutation restrictions are unchanged. See CRM_PROGRESS.md for
remaining modules and the hosted verification restriction for this stage.

Run `bun install`, `bun run test`, and `bun run lint`. Offline PostgreSQL tests use PGlite, apply the same SQL and emulate Supabase's auth roles; they never connect to a live project. Hosted Auth, PostgREST, cookies and real concurrent transactions still need checks against your selected test project after you authorize configuration. Validate owner/coordinator/viewer login, create → mark → undo, deadline refusal, manual review, cleanup twice, sign-out/back navigation and account switching.

### Vercel deployment target (manual, not performed)

1. Confirm an eligible **commercial Vercel plan** before production use; no purchase/upgrade is performed here. Do not use the separate portfolio Vercel project `dhruvrevops` or its repository `dhruvk1998/growthopsco-339bac08`.
2. In a trusted local/CI checkout, use Node 22 and the locked Bun dependencies: `bun install --frozen-lockfile`, `bun run test`, `bun run lint`, then **`bun run build:vercel`**. The Lovable environment runs its own preview build/typecheck automatically; that is not the production Vercel build.
3. `vite.vercel.config.ts` uses standard TanStack Start, React, Tailwind, path aliases and `nitro({ preset: "vercel" })`, preserving `src/server.ts` as the TanStack server entry. It does not import the Lovable preview preset. Nitro emits the Vercel Build Output API directory **`.vercel/output`** (static files, functions, `config.json`). Do not force a SPA output directory or add redirects; inspect actual output before deployment. Ignore generated `.vercel/` files in your own checkout; do not commit credentials/output.
4. When separately authorized to configure/deploy **this CRM repository only**, use checked-in `vercel.json`: framework `tanstack-start`, install `bun install --frozen-lockfile`, build `bun run build:vercel`. No Vercel project creation, repository linking or remote configuration is done by this code change.
5. Privately add the four runtime variables in Vercel, using the intended external Supabase connection and server-only privileged key. Preserve email/password Auth, no public signup and private owner/coordinator/viewer provisioning. Existing secure Lovable variables are not proof of Vercel configuration.
6. Check `.vercel/output/config.json`, static assets and server `.func` entry/runtime configuration. Confirm the custom SSR wrapper, content routes, server functions and protected cron route bundle correctly; scan client output for credential leakage without printing secrets.
7. After explicit deployment authorization, verify HTTPS cookies/login, owner/coordinator/viewer permissions, create → mark → undo, deadline refusal, all blockers, authorized cleanup twice and audit/queue readback, sign-out/account switching. An unauthorized GET must return 401 without privileged access. No destructive production check is authorized by this request.
8. Inspect Vercel's Cron Jobs view for exactly one daily `0 3 * * *` target, verify one real invocation and safe repeat/readback, and establish monitoring/manual retry after failure. Local configuration/tests do not prove hosted schedule execution.

See `DEPLOYMENT_REPORT.md` for exact local verification and remaining checks. No deployment/publishing has been performed by this change. Preserve backups until hosted permissions, links, audit and scheduling are verified.

### Official references (checked 2026-10-29)

- [TanStack Start hosting](https://tanstack.com/start/latest/docs/framework/react/guide/hosting)
- [Nitro Vercel preset](https://nitro.build/deploy/providers/vercel)
- [Vercel TanStack Start](https://vercel.com/docs/frameworks/full-stack/tanstack-start)
- [Vercel configuration schema](https://openapi.vercel.sh/vercel.json)
- [Cron security](https://vercel.com/docs/cron-jobs/manage-cron-jobs)
- [Cron usage, precision and plan limits](https://vercel.com/docs/cron-jobs/usage-and-pricing)
- [Cron behavior and failures](https://vercel.com/docs/cron-jobs)
- [Vercel Hobby non-commercial restriction](https://vercel.com/docs/plans/hobby)






Supabase: https://supabase.com/dashboard/project/biglmdldgoqojvxdhiiv
CRM: https://growthopscrm.lovable.app/login
