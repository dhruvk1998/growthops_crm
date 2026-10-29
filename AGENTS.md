<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

# GrowthOpsCo CRM rules
- Data goes through `LeadRepository` with an external Supabase implementation; ordinary access uses the caller's Supabase session and RLS, never a service-role client, to keep storage swappable and enforce permissions.
- Server-only modules (`*.server.ts`) are imported dynamically inside server-function/route handlers so DB credentials never reach the browser bundle.
- Pure deletion rules are tested in memory; authoritative atomic PostgreSQL RPCs enforce deadlines, audit events and blocker checks against direct API access and are tested in isolated PostgreSQL.
- Never fall back to localStorage as the database; when env is missing, show "not configured".
- Schema changes go in reviewed SQL under `supabase/migrations`; never apply them to live projects without user authorization.
- Supabase Auth uses request-scoped SSR cookie clients; authenticate with getUser and read roles from a separate protected table, preserving server-rendered route guards without browser-stored custom sessions.
- A request-keyed weak client cache and cookie overlay share refresh state only within one request; paginated reads with a stable ID tie-breaker prevent PostgREST row caps from hiding CRM records.
- Only the cron handler may dynamically load a service-role client after checking CRON_SECRET; cleanup locks linked tables and rows atomically to prevent concurrent blocker creation or duplicate audit events.
- Vercel production builds use the dedicated Vite config with TanStack Start and Nitro's Vercel preset; the Lovable config stays preview-only so deployment targets never conflict.
- Vercel Cron is the single scheduler and calls the existing protected cleanup GET route with its CRON_SECRET bearer token, avoiding duplicated database logic.
- Ordinary company/contact/service workflows use the allowlisted DirectoryRepository alongside LeadRepository with the same request-scoped caller client; this keeps storage swappable without opening generic lead/audit mutations.
- Directory CSV imports validate every row server-side and use one transactional insert with fresh IDs; this prevents partial imports and accidental overwrites, while exports escape spreadsheet formulas.
- Deals, activities, tasks, proposals and projects use a separate allowlisted WorkflowRepository on the same caller client; foreign keys are checked through visible records and activity authors are server-assigned, without exposing any lead/audit/cleanup mutation.

- Project tasks remain in the tasks workspace with a project filter and prefilled project link; proposal status stays validated text because the existing schema has no enum, avoiding invented schema constraints.
