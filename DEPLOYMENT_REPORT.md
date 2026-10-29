# GrowthOpsCo CRM — Vercel deployment target

## Result and scope

This CRM project **15e46350-c44d-4349-a1c4-a05a42df1592** now targets **Vercel** through an isolated TanStack Start + Nitro production configuration. Lovable preview keeps its existing configuration. No deployment/publishing, Vercel project creation/change, GitHub linking/repository change, secret change, purchase/upgrade, Supabase settings/data change or Lovable Cloud provisioning occurred. The separate portfolio app/project `dhruvrevops` and repository `dhruvk1998/growthopsco-339bac08` were not accessed.

**Vercel Hobby is personal/non-commercial only. This business CRM requires an eligible commercial plan for production publishing.** This is deployment-ready source configuration, not a claim of verified production packaging or hosted operation.

External Supabase integration remains unchanged: prior connection to “Growthops CRM” (`biglmdldgoqojvxdhiiv`), email/password Auth, private provisioning, separate roles and RLS. Browser setup currently shows all four variables set; this proves presence only, not Auth/schema health. No credential values were printed or added to code/config/docs.

## Production configuration evidence

- Installed versions remain TanStack Start **1.168.60**, Vite **8.1.5**, Nitro **3.0.260603-beta**. No framework migration or Nitro version change was made.
- `vite.vercel.config.ts` imports standard Vite plugins and `nitro` from `nitro/vite` with explicit `preset: "vercel"`. It preserves TanStack's `server: { entry: "server" }`, hence the existing `src/server.ts` default fetch/error wrapper. It never imports the Lovable preview config or adds a second adapter.
- Vite production **configuration resolution passed** with Nitro plugins and the Vercel preset reporting Node **nodejs22.x**, web entry format. Client output resolves to `.vercel/output/static`; SSR service is registered separately. Nitro builder configuration also resolved output directory `.vercel/output`, server directory `.vercel/output/functions/__server.func`, static directory `.vercel/output/static`.
- Official Vercel JSON schema fetched on 2026-10-29 explicitly includes framework identifier **`tanstack-start`**. `vercel.json` sets it and overrides install/build commands to locked Bun installation and `bun run build:vercel`. No guessed outputDirectory override or SPA redirect is added.
- Configuration resolution is **not** compilation, SSR packaging, artifact verification or proof of custom-entry execution. Nitro is the existing prerelease; its exact production artifact must be validated before reliance.
- Netlify-only configuration, dispatcher, direct packages and tests were removed. Nitro upstream optional peer metadata still mentions `@netlify/runtime` and `@netlify/blobs` in `bun.lock`; these are not CRM deployment dependencies. Upstream package metadata was not falsified.

## Cleanup and security

One Vercel cron declaration calls **GET** `/api/public/cron/purge-non-workable` with expression **`0 3 * * *`**, UTC. Vercel sends the production `CRON_SECRET` as `Authorization: Bearer <CRON_SECRET>`. The existing handler checks it before dynamically importing the server-only service-role client and calling the same atomic PostgreSQL RPC. POST remains protected for authorized manual retries; no dispatcher or second schedule remains.

Jobs run only on production deployments. **Hobby can invoke any time 03:00–03:59 UTC**, not exactly 03:00; its daily-only technical support does not make it eligible for commercial use. Pro/Enterprise have per-minute precision, not an exact execution or delivery guarantee. Cron consumes Functions usage and inherits the selected plan's limits; no specific duration/uptime guarantee or purchase is assumed. Vercel does not automatically retry failed cron jobs. Inspect queue/audit after failures/timeouts before an authorized retry; an in-flight transaction may still finish.

SQL and Auth/repository code were not changed. Deadlines remain exactly **168 hours / 604800000 milliseconds**, undo only before deadline, Unresponsive distinct, overdue-only idempotent cleanup, transaction/linked-table/row locks, minimal non-PII audit, lead-exclusive tasks/activities only, companies/contacts retained, active/won deals/active projects/invoices/payments flagged for manual review. Physical removal occurs on the next successful job, not at the precise database deadline.

## Exact changed files

Added:
- `vite.vercel.config.ts`
- `vercel.json`
- `src/test/vercel-deployment.test.ts`

Updated:
- `package.json`, `bun.lock` — replace production command; remove direct `@netlify/functions`, `@netlify/vite-plugin-tanstack-start`, `@iarna/toml` dependencies.
- `tsconfig.json` — check isolated Vercel config, remove old dispatcher include.
- `eslint.config.js` — exclude generated Vercel instead of old host output.
- `vite.config.ts` — preview-only comment.
- `.env.example` — Vercel runtime/cron guidance, no secret values.
- `src/routes/api/public/cron/purge-non-workable.ts` — host comment only; handler unchanged.
- `src/test/cron.test.ts` — Vercel-configured GET → existing RPC test, protected POST preservation and environment cleanup.
- `src/routes/setup.tsx` — Vercel Cron wording only.
- `README.md`, `SETUP.md`, `AGENTS.md`, `MIGRATION_REPORT.md`, `DEPLOYMENT_REPORT.md`, `roadmap.md` — hosting, security, commercial eligibility and verification guidance.
- Project memory — user's Vercel target, scope and schedule requirements supersede prior hosting preference.

Removed:
- `vite.netlify.config.ts`
- `netlify.toml`
- `netlify/functions/purge-non-workable.ts`
- `src/test/netlify-schedule.test.ts`

No schema migration or CRM behavior change. Root `.gitignore` was not changed; ignore generated `.vercel/` output in your trusted checkout before building and never commit its credentials/output.

## Exact verification results (2026-10-29 UTC)

- **Full tests:** `bun run test`, exit **0**, **60 tests passed / 8 files**, start **22:09:35**, duration **31.08 seconds**. Includes 20 PostgreSQL migration/RLS/lifecycle tests in isolated PGlite, domain/auth/repository/routing tests, Vercel deployment rules, configured cron GET to the atomic RPC and protected POST. No remote SQL/data operation.
- **Full lint:** `bun run lint`, exit **1**, **673 errors and 6 warnings**. All errors are existing formatting violations in generated `src/integrations/supabase/{auth-middleware.ts,client.server.ts,client.ts,previewAuthStorage.ts,types.ts}`; six existing shared-UI fast-refresh warnings. These files are outside hosting scope and were deliberately not rewritten. Full lint is not claimed clean. Exact remaining check: format/review those generated files in a separately authorized cleanup, then rerun `bun run lint`.
- **Changed-file lint:** `bunx eslint vite.vercel.config.ts vite.config.ts src/test/vercel-deployment.test.ts src/test/cron.test.ts src/routes/setup.tsx src/routes/api/public/cron/purge-non-workable.ts eslint.config.js`, exit **0**.
- **Automatic Lovable preview build/typecheck:** latest observed **build OK at 22:09:27 UTC**. Earlier transient missing removed-package errors were followed by successful builds. Manual builds/typechecks are prohibited by the environment and were not duplicated.
- **Production Vercel config and Nitro output resolution:** passed, including explicit Node runtime/output directories above. One non-failing Vite paths-plugin deprecation advisory remains.
- **Browser:** `/setup` at 1280×1800 shows four settings set, new Vercel Cron wording and no page errors; screenshot inspected. Live unauthenticated cron **GET returns 401**. No authorized live cleanup or record mutation was attempted.
- **Metadata/security review:** all content routes retain unique app-specific title/description/Open Graph metadata and twitter card/type. Service-role import boundary and SQL remain unchanged. No active old hosting references in app routes/setup/config/README/AGENTS; old-host names remain only in removal assertions/report history and upstream optional peers.
- `git diff --check`, exit **0**; managed Git supplied no useful tracked stat, so exact inventory and direct review document scope.
- **Not run:** full `bun run build:vercel`, artifact/client-secret scan, packaged SSR/function execution, hosted Auth/RLS/session flow, multi-connection concurrency or actual Vercel cron execution. `.vercel/output/config.json` is not generated. Config resolution does not substitute for any of these checks.
- An optional Python JSON-schema validation helper could not run because `jsonschema` is unavailable; framework identifier was checked directly against the fetched official schema, and deployment rules are tested. An initial Nitro builder import used the wrong export and was corrected to `nitro/builder`; output resolution then passed.

## Exact remaining checks and user-controlled steps

**Later CRM stage (2026-10-29):** the five generated integration files listed above
have now received formatting-only cleanup, superseding the historical lint failure.
Current full-suite/lint/preview results are in CRM_PROGRESS.md. Vercel config and cron
were preserved; no production build, deployment or remote data/settings changes ran.

1. In a trusted local/CI checkout: **`bun install --frozen-lockfile && bun run build:vercel`**. Inspect `.vercel/output/config.json`, static files, `functions/__server.func/.vc-config.json` and the referenced handler for Node runtime/SSR/route packaging; scan client assets for private-key leakage without logging credentials. Validate the custom SSR wrapper and server functions. This environment only runs the ordinary preview build/typecheck automatically, not this production command.
2. Confirm a commercial-eligible Vercel plan; do not purchase/upgrade or deploy without separate authorization. Configure only this CRM repository/project, never the portfolio project/repo named above.
3. Privately configure `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, server-only `SUPABASE_SERVICE_ROLE_KEY`, `CRON_SECRET` in Vercel Production settings; Lovable connection does not copy them to Vercel. No secrets in `vercel.json` or untrusted previews. Preserve existing external Supabase Auth/RLS settings; do not reapply its migration.
4. After authorization, verify hosted HTTPS sign-in/session/role restrictions, lead lifecycle/deadline/blockers, authorized cleanup twice and audit readback against approved test data, plus a real production cron invocation. Inspect Functions duration limits and Cron Jobs view for exactly one daily target; monitor failure/delayed cleanup.

Official documentation links and fresh-setup instructions are in `SETUP.md`. Nothing is deployed or purchased by this change.
