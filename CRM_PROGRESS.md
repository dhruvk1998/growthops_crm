# GrowthOpsCo CRM — bounded completion stage, 2026-10-29

## Delivered (not a claim the app is complete)

- Companies: real caller-session Supabase reads, create/edit, URL search/client filter,
  record details and company-contact navigation.
- Contacts: reads/create/edit, email/phone validation, optional existing-company selection,
  company filter and linked company details.
- Services: reads/create/edit, description, nonnegative price and unit validation.
- CSV for these three modules: templates, export filtered readable rows with spreadsheet
  formula escaping, 1 MB/500-row validation, row errors, explicit import confirmation,
  server revalidation and a single transactional INSERT with fresh IDs. No upserts/seeds.
- CSV sidebar now links to supported record workspaces; remaining-module CSV is not done.
- Initial reads prime Query cache under the existing cookie-auth route guard and use
  Suspense; module loading/error/retry/empty states are present. Existing auth is unchanged.
- Viewer reads/exports only; coordinator/owner create/edit/import; writes reauthorize every
  request. No new deletion path. Existing owner-only ordinary deletion policies remain
  unchanged; no leads/disposition/audit writes are exposed by this module repository.
- Formatting-only cleanup of exactly the five generated Supabase integration files.

## Exact files

Added:
- `src/crm/directory.ts` — allowlisted data contract and validation.
- `src/crm/directory-csv.ts` — shared CSV parsing/export.
- `src/crm/directory.functions.ts` — authenticated server reads/mutations/import.
- `src/crm/directory.queries.ts` — Query cache options.
- `src/crm/repo/directory.server.ts` — caller-scoped Supabase repository.
- `src/components/directory-workspace.tsx` — lists, details, editors and CSV dialogs.
- `src/crm/directory.test.ts`, `src/crm/directory.functions.test.ts`,
  `src/crm/repo/directory.test.ts`, `src/crm/repo/directory-sql.test.ts` — offline tests.
- `CRM_PROGRESS.md` — this report.

Updated:
- `src/crm/context.server.ts` — supplies directory repository with the same caller client.
- `src/routes/modules.$module.tsx` — implemented module screens, guard/loading and CSV hub.
- `src/components/app-shell.tsx` — default module URL search parameters only.
- `package.json`, `bun.lock` — Papa Parse and its types (CSV library); hosting preserved.
- `src/integrations/supabase/auth-middleware.ts`, `client.server.ts`, `client.ts`,
  `previewAuthStorage.ts`, `types.ts` — Prettier formatting only, no behavior changes.
- `README.md`, `SETUP.md`, `AGENTS.md`, `MIGRATION_REPORT.md`, `DEPLOYMENT_REPORT.md`,
  `roadmap.md` — current stage, architecture boundaries, remaining work and lint correction.

No SQL migration was needed or changed. No remote SQL/data writes, Supabase settings,
secrets, GitHub/Vercel accounts/projects, deployment, production build or portfolio access.
`vercel.json`, production Vite config, protected cron, lead lifecycle and Auth/RLS unchanged.

## Verification

- Full tests: `bun run test`, exit **0**, **89 passed / 12 files**;
  final run 2026-10-29 **22:25:39 UTC**, **32.55 seconds**. Existing 60 tests remain passing.
- Full lint: `bun run lint`, exit **0**, **0 errors / 6 existing shared-UI fast-refresh warnings**.
- Automatic preview build/typecheck: **build OK**, observed **22:25:33 UTC**
  after the own-property module guard change.
- Added tests cover viewer denial, owner/coordinator writes, signed-out reads, strict
  allowlist/unknown-field rejection, CSV validation/formulas, server create/edit/readback,
  pagination, company-link rejection, single-insert behavior and uncapped import counts.
- Local PGlite applies the existing migration and verifies company/contact/service
  persisted writes/readback, relationship FK enforcement and unchanged RLS role matrix.
  Existing seven-day deadline/blockers/audit/atomic rollback/idempotency tests still pass.
- Independent source review found no service-role or lead/audit bypass; caught inherited
  module names and the guard now uses `Object.hasOwn`.
- Browser: all four implemented module/CSV URLs redirect signed-out visitors to `/login`
  retaining intended destination. `/setup` shows four variables set, no page errors;
  desktop 1280×1800 and mobile 390×844 inspected, no mobile page-width overflow.
- **Hosted signed-in create/edit/import/readback was not run.** Browser auth is
  `external_unmanaged`, and this request forbids altering external Supabase data.
  Mocked/offline persistence tests are not hosted end-to-end verification.
  Exact remaining check: with separately authorized test data and a provisioned account,
  create a company → linked contact → edit/readback → service → CSV import/export,
  then confirm viewer denial and coordinator/owner allowed writes through the actual UI.
- No manual typecheck/build or production Vercel build was attempted; preview checks are automatic.

## Historical Stage 1 remaining checklist

- Deals pipeline tied to existing leads/companies without opening disposition updates.
- Activities/call logs and follow-up tasks with links/assignees.
- Proposals and projects/project tasks.
- Invoices/payments with linked details and validations.
- Expanded dashboard summaries and safe self-profile/configuration settings.
- CSV for remaining ordinary modules, and hosted authorized end-to-end checks.

Remaining placeholder modules deliberately retain “Not built yet”; this stage does not
rename placeholders to pretend they are working. Implementation is staged because the
full remaining scope exceeds one bounded edit; no credentials block further source work.
Hosted writes wait on separate authorization/test account, and Vercel packaging/deployment
remain outside this edit. Keep these tasks open in `roadmap.md`.

## Stage 2 delivered — 2026-10-29

- `/modules/deals`: real caller-scoped deals, board/list, open/won/lost stage controls,
  title/value/lead/company create/edit, linked details, URL search/filter/view selection.
  Stage changes affect deals only; they never modify the linked lead's status/disposition.
  Reopening a won/lost deal to open is supported by the existing schema.
- `/modules/activities`: persisted call/email/meeting/note logs with summary/time,
  existing lead/deal/contact links, search/type filter, details/create/edit. Existing other
  kind values can be retained on edit. Actor is assigned by the server on creation;
  the edit path preserves the original actor and rejects actor_id input.
- `/modules/tasks`: persisted title/due date/completion, existing lead/deal/project links,
  assignee selection, search/open/completed/overdue/assignee filters, details/create/edit,
  wired complete/reopen controls. Local date inputs are converted to UTC before validation.
- All mutations re-check owner/coordinator authority. Viewer receives read-only views.
  Every non-null FK is checked against caller-visible records, assignees against visible
  enabled profiles, and database FK/RLS remains authoritative at write time.
- Existing nullable independent links are retained: a record may have multiple parents
  or none. This stage does not invent cross-parent consistency constraints. Existing
  deletion-queue leads remain readable/linkable and marked in selectors; no lead mutations.
- No delete buttons/functions, lead/disposition/audit writes, cleanup changes, service-role
  ordinary access, migrations, remote SQL/data writes, secrets, provisioning, settings,
  repository/account integration changes or deployments. Vercel/Nitro config unchanged.

### Stage 2 exact changed files

Added:
- `src/crm/workflows.ts` — strict allowlisted schemas, repository contract and field/link metadata.
- `src/crm/workflows.functions.ts` — validated authenticated list/link/save server functions.
- `src/crm/workflows.queries.ts` — initial Query cache options.
- `src/crm/repo/workflows.server.ts` — paginated caller-scoped reads, link checks and saves.
- `src/components/workflow-workspace.tsx` — board/list/detail/edit/create/filter controls.
- `src/crm/workflows.test.ts`, `src/crm/workflows.functions.test.ts`,
  `src/crm/repo/workflows.test.ts`, `src/crm/repo/workflows-sql.test.ts`,
  `src/components/workflow-workspace.test.tsx` — domain/server/repository/PostgreSQL/UI tests.

Updated:
- `src/crm/context.server.ts` — supplies workflow repository using existing caller client.
- `src/routes/modules.$module.tsx` — Stage 2 loading/rendering and URL view/assignee params.
- `src/components/app-shell.tsx` — Deals pipeline navigation.
- `AGENTS.md`, `roadmap.md`, `CRM_PROGRESS.md` — boundaries, progress and limitations.
- `src/integrations/supabase/previewAuthStorage.ts`, `types.ts` — formatting only:
  these two generated files were regenerated since Stage 1 and again caused lint failure;
  Prettier restored formatting without behavior changes. No other integration changes.

### Stage 2 exact verification

- **Full tests:** `bun run test`, exit **0**, **122 passed / 17 files**, final run
  **22:36:19 UTC**, **43.09 seconds**. All prior 89 tests remain passing.
- **Full lint:** `bun run lint`, exit **0**, **0 errors / 6 existing fast-refresh warnings**.
- **Automatic preview build/typecheck:** observed **build OK at 22:35:54 UTC**.
  Manual builds/typechecks and production builds were not attempted.
- Offline tests cover strict inputs/stages/dates/UUIDs, viewer denial per table,
  owner/coordinator writes, signed-out reads, caller-author assignment, link failures,
  pagination, FK race failures, task completion payload, form submission/error handling,
  actual local PostgreSQL FK/RLS/persistence, unchanged lead status after deal changes,
  direct lead/audit write denial, exact **604800000 ms** deadline, expired recovery denial,
  linked deal/project cleanup blockers and idempotent repeat. Existing lifecycle suite passes.
- Browser checked all Stage 2 URLs signed-out: each redirects to `/login` preserving
  destination, Deals navigation present, no page errors, mobile setup no width overflow.
- Hosted signed-in writes/readback were **not run**: external unmanaged auth and explicit
  no-remote-data-write scope. Offline tests do not prove hosted persistence or full signed-in UI.
  Next authorized check: create linked deal → log call → assigned task → edit/complete/reopen
  → reload/readback for owner/coordinator and confirm viewer denial using approved test data.
- Existing schema allows updating actor_id through direct owner/coordinator Data API access;
  this stage exposes no such path and cannot promise DB-enforced immutable activity authors.
  Assignees are visible enabled profiles, not role-filtered teammates: existing role RLS exposes
  only the caller's own role row, so broader role filtering requires a separately reviewed change.
  Neither RLS nor schema was altered to conceal these limitations.

### Remaining after Stage 3A

Invoices/payments, expanded dashboard, safe profile/settings controls and extended CSV
remain unfinished. Proposals and project management are delivered in Stage 3A below.
Production artifacts/hosted cron and authenticated writes still need separate authorization.

Revision reporting uses the available managed checkout HEAD, not a manually created commit;
the final chat identifies the observed SHA. Stateful Git operations were not performed.

## Stage 3A delivered — 2026-10-29

- Proposals: persisted list/detail/create/edit, search across fields and deal labels,
  status filter, title, free-text status, nonnegative finite total, existing-deal linking.
  The schema has no proposal-status enum: text is trimmed, required and bounded to 80
  characters rather than inventing a closed status list. No line items or document/PDF
  generation: the existing table has only title/status/total/deal/date/ID.
- Projects: persisted list/detail/create/edit, search, active/completed/cancelled filter,
  company/deal links, and a Project tasks action opening the existing tasks workspace.
- Tasks: project filter, project detail navigation, and prefilled project on new tasks
  in a selected project. Existing assignees, due date, edit, complete/reopen and other
  task filters remain integrated. Completing a project does not silently complete tasks.
- Reused the existing WorkflowRepository/functions/Query/workspace instead of adding
  a second ordinary-write boundary. Newly allowlisted proposals/projects use the same
  caller-scoped pagination, validation, visible-FK checks, fresh-ID inserts and ID-scoped
  edits. Owner/coordinator writes; viewer read-only. No delete path.
- No migration, schema, live write, SQL, Auth/RLS/settings, provisioning, secrets,
  hosting/deployment or account/project changes. Lead/audit/cleanup functions and
  Vercel/TanStack/Nitro configuration are untouched. Cross-parent consistency is not
  invented: nullable independent company/deal/project links follow the existing schema.

### Stage 3A exact changed files

Added:
- `src/crm/stage3a.test.ts` — strict proposal/project input regressions.
- `src/crm/repo/stage3a-sql.test.ts` — local PostgreSQL persistence, role/FK/lead regressions.

Updated:
- `src/crm/workflows.ts` — proposal/project allowlist, schemas, fields and labels.
- `src/components/workflow-workspace.tsx` — proposal/project forms, details and filters;
  linked project navigation, project task filter and prefill.
- `src/routes/modules.$module.tsx` — validated project search parameter; existing loader
  and rendering automatically include newly allowlisted modules.
- `src/crm/workflows.functions.test.ts` — per-kind role and signed-out denial.
- `src/crm/repo/workflows.test.ts` — proposal/project insert/edit/link checks.
- `src/components/workflow-workspace.test.tsx` — proposal/project submissions, project
  task prefill, viewer controls; offline ResizeObserver support for task editor.
- `src/integrations/supabase/previewAuthStorage.ts`, `types.ts` — formatting only after
  generated-file formatting regressed again; no behavior change.
- `AGENTS.md`, `roadmap.md`, `CRM_PROGRESS.md` — current architecture/progress/limitations.

### Stage 3A exact verification

- Full suite: `bun run test --maxWorkers=2`, exit **0**, **149 passed / 19 files**,
  final run **22:51:04 UTC**, **37.31 seconds**. All previous 122 tests pass.
  Initial default parallel run had one existing local PostgreSQL test hit its 5-second
  timeout under contention; bounded concurrency passes without changing assertions.
- Full lint: `bun run lint`, exit **0**, **0 errors / 6 existing shared-UI warnings**.
  Generated-file formatting restored full lint; no lint exclusions added.
- Automatic Lovable preview build/typecheck: **build OK at 22:51:48 UTC** after resolving
  a new mock typing error. No manual build/typecheck or Vercel packaging performed.
- Local PGlite runs the unchanged complete migration: proposal/project create/edit/readback,
  linked project tasks, owner/coordinator access, viewer/anonymous denial, coordinator
  deletion denial, missing deal/company/project/assignee FKs, project-status CHECK,
  unchanged linked lead status, direct lead/audit-write denial, exact **604800000 ms**
  non-workable deadline, expired undo denial, active-project cleanup blocking even when
  its linked deal is lost, retained linked tasks/proposals and repeat-cleanup idempotency.
- Offline UI tests exercise real workspace forms and mutation payloads, including task
  project prefill; mocked saves are not hosted persistence checks.
- Browser: proposal/project/project-filtered task URLs redirect signed-out visitors to
  `/login`, preserving destination. Desktop 1280×1800/mobile 390×844 sign-in inspected;
  no mobile width overflow or page errors. Signed-in module appearance was not verified
  in the hosted browser; no authenticated hosted writes were performed.
- Hosted signed-in create/edit/reload/readback, role-matrix UI checks and project-task
  navigation still require separately authorized test accounts/data. No credentials
  were accessed or requested. Vercel packaging, cron execution and production deployment
  remain unverified/outside scope.
- Observed managed checkout HEAD at **22:52 UTC**:
  `2eaf8fd2d4bcd6bec0e5ba7c34b08f4770fcd933`.
  This is an observed revision, not a manually created commit or proof of publishing.
