# GrowthOpsCo CRM — external Supabase migration report

> Current hosting: Vercel via isolated TanStack Start + Nitro production configuration and one protected GET cron (`0 3 * * *`, UTC). Hobby may invoke during 03:00–03:59 and is personal/non-commercial only; this business CRM needs an eligible commercial plan. See `DEPLOYMENT_REPORT.md` and `SETUP.md`. The migration results below are historical; prior external connection is to “Growthops CRM” (`biglmdldgoqojvxdhiiv`), not Lovable Cloud. This hosting change does not touch remote settings/data or deploy.

## Result and scope

The existing CRM code now targets an external Supabase project with Supabase Auth, PostgreSQL RLS and transactional deletion RPCs. No Lovable Cloud provisioning, external project connection/selection, live SQL, account creation, record import/deletion, deployment or publishing was performed. No separate portfolio project was accessed.

Preserved actual features: dashboard metrics; lead creation; list and Kanban pipeline; search/filter/status changes; non-workable reason and optional note; deletion queue; undo; profile/role-driven access; sign-in/sign-out; setup indicators; daily cron URL and schedule. At migration time the other modules were placeholders. Company/contact/service workflows and their CSV support were added in a later bounded stage using the existing schema unchanged; other modules remain unfinished. See CRM_PROGRESS.md and roadmap.md for current status.

## Exact files added

- `.env.example`
- `MIGRATION_REPORT.md`
- `roadmap.md`
- `supabase/migrations/20261029203500_growthops_crm.sql`
- `src/crm/context.server.ts`
- `src/crm/auth/supabase.server.ts`
- `src/crm/auth/admin.server.ts`
- `src/crm/auth/session.test.ts`
- `src/crm/auth/supabase-client.test.ts`
- `src/crm/repo/supabase.server.ts`
- `src/crm/repo/supabase.test.ts`
- `src/crm/repo/supabase-sql.test.ts`
- `src/test/cron.test.ts`

## Exact files updated

- `.gitignore` — ignore private env files, retain blank example.
- `AGENTS.md` — external storage/auth/security architecture rules.
- `SETUP.md` — external credentials, manual schema/account setup, cutover and verification steps.
- `package.json` — replace database dependencies; remove old migration/seed/provision commands.
- `bun.lock` — updated dependencies.
- `src/crm/crm.functions.ts` — Supabase config, Auth and repository-backed CRM commands.
- `src/crm/auth/session.server.ts` — Supabase Auth replaces custom sessions/password verification.
- `src/crm/domain/repository.ts` — document Supabase storage boundary.
- `src/crm/domain/deletion.ts` — no undo after any deadline; only system purges; remove non-null assertions.
- `src/crm/domain/deletion.test.ts` — align system-only purge and remove non-null assertions.
- `src/crm/repo/memory.ts` — retain contact-shared activities.
- `src/components/app-shell.tsx` — clear private queries on sign-out; mobile sign-out remains available.
- `src/components/leads/non-workable-dialog.tsx` — exact-duration wording; physical cleanup timing stated accurately.
- `src/components/not-configured.tsx` — formatting only.
- `src/routes/__root.tsx` — formatting only.
- `src/routes/index.tsx` — complete leaf metadata and formatting.
- `src/routes/leads.tsx` — complete leaf metadata and formatting.
- `src/routes/deletion-queue.tsx` — complete leaf metadata and formatting.
- `src/routes/login.tsx` — router navigation after login; complete metadata and formatting.
- `src/routes/setup.tsx` — Supabase variable indicators; complete metadata.
- `src/routes/modules.$module.tsx` — complete leaf metadata and formatting.
- `src/routes/api/public/cron/purge-non-workable.ts` — secret verification before service-role import; atomic RPC, sanitized errors/counts.
- `src/test/setup.ts` — support isolated PostgreSQL tests without browser globals.

## Exact files removed

- `src/crm/repo/libsql.server.ts`
- `src/crm/repo/libsql.test.ts`
- `src/crm/repo/schema.ts`
- `src/crm/auth/password.ts`
- `scripts/migrate.ts`
- `scripts/seed.ts`
- `scripts/provision-user.ts`

Project memory was updated to replace the earlier storage prohibition with the user's external-Supabase requirements. `vercel.json`, Vite hosting configuration, and portfolio files were not changed.

## Security and deletion review

- 14 namespaced CRM tables, explicit privilege resets/grants, RLS on every table, no anonymous CRM access.
- Separate protected roles; signup metadata never grants CRM authority. All members read; owner/coordinator edit; viewer cannot write. Profile updates cannot change disable/role state.
- Public-key caller sessions serve ordinary CRM access; service-role client is loaded only after cron-secret verification.
- HTTP-only Secure SameSite=Lax Supabase cookies, Auth-server `getUser()` validation, private/no-store responses, one client and refreshed cookie overlay per request.
- Exact 168-hour deadline (604800000 ms); database time authoritative. Undo denied at/after deadline, including manual-review leads.
- Atomic mark/recover/cleanup and audit insertion; linked-table locks and lead row locks prevent blocker-check/delete races.
- Active/won deals, active projects, invoices and payments block purge and flag manual review.
- Only lead-exclusive tasks and activities removed. Contact/deal/project-shared records survive with lead references detached. Companies/contacts untouched.
- Audit receives only IDs, timestamps, status/reason codes and blocker counts. Notes and personal details never copied into audit.
- Daily `0 3 * * *` UTC schedule and GET/POST cron endpoint retained. This is not an exact 03:00 execution guarantee; Hobby is hourly. Repeated cleanup is idempotent.
- Explicit pagination and stable ID ordering prevent Supabase's API row cap from silently hiding leads; related names fetched by referenced IDs.

Independent reviews identified and resolved same-request login verification, implicit API truncation, and inherited default database privileges. Offline tests cover these fixes.

## Verification results

- **Full suite: 55 tests passed in 7 files.** Includes 20 real PostgreSQL migration/RLS/lifecycle tests in isolated PGlite, 20 domain tests, 5 session tests, 3 cookie/config tests, 2 repository tests, 4 cron-auth tests and 1 route test.
- PostgreSQL tests also use permissive default grants to verify the migration revokes unintended access. Atomic rollback tested by forcing an audit failure.
- **Lint: exit 0, no errors; 6 existing fast-refresh warnings** in shared UI components.
- **Harness typecheck/build: latest `build OK` at 2026-10-29 20:45:05 UTC.** Manual build/typecheck was not duplicated; the environment runs them automatically.
- Browser: `/`, `/leads`, `/deletion-queue`, `/setup`, `/login` render in the real unconfigured state; sign-in disabled; setup labels show missing Supabase variables; no page errors. Desktop 1280×1800 and mobile 390×844 setup screenshots inspected; no mobile page-width overflow.
- Unconfigured cron returns 503. Isolated tests verify invalid authorization returns 401 without creating a privileged client and authorized cleanup returns counts only.
- Obsolete-reference scan: **no old database references in app source, direct dependencies, configuration example or setup/architecture docs.** The lockfile still contains `@libsql/client` only as an upstream **optional peer declaration inside Nitro's `db0` dependency**; it is not installed as an app dependency or used by CRM code. Do not manually falsify upstream package metadata to remove that text.
- Source-level secret/boundary review passed. Automatic bundling succeeded; a separate production artifact/secret scan was not available in this checkout. No real private keys were supplied, exposed, logged or added to source.
- Git diff commands produced no useful tracked-file delta in this managed environment. File inventory, source review, test output and independent review were used instead; whitespace check reported no errors.

## Required user actions / blockers

1. Prior work connected external “Growthops CRM” (`biglmdldgoqojvxdhiiv`); this hosting change does not alter the connection. Historical migration checks below are not current hosted verification.
2. The user reports the migration already applied; do not reapply. Schema steps in SETUP.md are for fresh authorized setup, never the portfolio project.
3. Set `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY` (anon or public key), `SUPABASE_SERVICE_ROLE_KEY` (server only), and `CRON_SECRET`. No `VITE_*` variables or paid Lovable Cloud database are required.
4. Privately create/confirm Auth accounts, set profile names, and assign owner/coordinator/viewer roles with the documented SQL. Enable email/password and disable public signup in your own Auth settings; hosted settings were not inspected.
5. For existing data, perform a separate backed-up cutover with old-to-new user UUID mapping and validation. No records or legacy passwords/sessions were migrated.
6. After authorizing test-project configuration, verify live Auth/PostgREST cookies, signed-in lead create → mark → undo, expired recovery refusal, each blocker, cleanup twice, and sign-out/account switching. Current external configuration is the blocker to these checks.
7. Validate your Vercel hosting adapter and cron execution separately. Scheduling declaration is preserved, not proven live. No publishing/deployment has occurred.

PGlite validates actual PostgreSQL SQL/RLS but does not emulate hosted Auth/PostgREST or prove multi-connection lock behavior. Real concurrency, production bundles, host integration and scheduled invocation remain unverified. Manual-review rows are retained for investigation; no new review-resolution UI was added. Overdue rows remain visible in the queue until cleanup, matching the previous implemented queue behavior; physical removal occurs on the next successful scheduled run.
