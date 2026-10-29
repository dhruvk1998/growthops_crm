# External Supabase migration
- [x] Inspect existing features and security boundaries.
- [x] Replace storage/auth/configuration and remove former database implementation.
- [x] Provide reviewed SQL schema, RLS, roles and atomic cleanup safeguards.
- [x] Run tests, lint, preview checks and security/regression review.
- [x] Document changed files, results and external setup blockers.
- [x] Handle unconfigured cleanup URL navigation without a preview error; preserve machine-call failure status.
- [ ] Verify live external Auth, persisted UI flows and concurrent cleanup — external connection was completed previously; writes/cleanup testing awaits separately authorized test data/accounts.

# Vercel hosting change
- [x] Verify supported TanStack Start + Nitro Vercel production configuration, isolated from Lovable preview (configuration resolution, not full build).
- [x] Replace hosting files/dependencies and test single protected daily GET cron; preserve Supabase and deletion safeguards.
- [x] Update hosting/security/setup reports and run tests, lint, config/artifact checks and automatic preview checks; document failed full lint and unavailable artifact.
- [x] Resolve existing generated Supabase formatting errors and rerun full lint — formatting-only cleanup; no behavior changes.
- [ ] Verify production artifact and hosted schedule — blocked by automatic-build-only environment and user-controlled commercial-plan/deployment authorization.

# Persisted CRM completion — staged scope
- [x] Stage 1: companies, contacts linked to companies, and service catalogue: caller-session/RLS-backed list/detail/create/edit, search/filter, role checks and validation; hosted write verification remains separately blocked below.
- [x] Stage 1: validated CSV import/export for companies, contacts and services; row-level errors and no fake data.
- [x] Stage 1: test server/repository/CSV flows, full tests/lint and automatic preview checks; exact results in CRM_PROGRESS.md.
- [x] Stage 2: deals pipeline and relationships to existing leads/companies; no lead/disposition writes. Source and offline verification complete; hosted writes remain unverified.
- [x] Stage 2: activities/call logs and follow-up tasks with visible record links and enabled-profile assignments; source and offline verification complete. Results/limitations in CRM_PROGRESS.md.
- [x] Stage 3A: proposals and projects with integrated project-task workflows; 149 tests/19 files pass, full lint 0 errors/6 warnings, automatic preview build/typecheck OK. Exact files/results/limitations in CRM_PROGRESS.md; hosted signed-in writes remain unverified.
- [ ] Stage 3B: invoices and payments with linked details and validated workflows.
- [ ] Stage 3: dashboard summaries across modules and safe self-profile/configuration settings (no invented organization settings).
- [ ] Stage 3: extend CSV support to remaining ordinary modules and verify signed-in persisted flows against separately authorized test data/accounts.
No remote writes, settings changes, secrets, deployments or production builds are authorized in this stage. Remaining stages are not complete or running in the background.
