# GrowthOpsCo CRM

Internal CRM with external Supabase Auth/PostgreSQL. The deployment target is
**Vercel** using TanStack Start with Nitro's Vercel output. Vercel Hobby is for
personal, non-commercial use only; this business CRM needs an eligible commercial
plan before production publishing. Nothing has been deployed or purchased.

Read [SETUP.md](SETUP.md) for manual setup and [DEPLOYMENT_REPORT.md](DEPLOYMENT_REPORT.md)
for compatibility evidence, cron limits and verification gaps.

## CRM completion status

Companies, company-linked contacts and services now have caller-session/RLS-backed list,
detail, create/edit and validated CSV import/export screens. No sample rows are seeded.
Lead screens and deletion safeguards are unchanged. Activities, tasks, proposals,
projects, invoices, payments, expanded dashboard and profile settings remain unfinished;
see [roadmap.md](roadmap.md) and [CRM_PROGRESS.md](CRM_PROGRESS.md). This app is not complete.

This project was built with [Lovable](https://lovable.dev).

## Build with Lovable

Open your project in the [Lovable editor](https://lovable.dev) and keep building.

- **Ship faster**: describe what you want to build and Lovable handles the code.
- **Stay in sync**: connect the project to GitHub and every change made in Lovable is committed straight to your repository.
- **Full ownership**: this code is yours. Push to your repository and your changes sync back into Lovable, ready for your next prompt.

## Development

Prefer working locally? You need Node.js and npm — [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating).

```sh
git clone <this-repository-url>
cd <repository-name>
npm i
npm run dev
```

## Built with

- TanStack Start
- TypeScript
- React
- Tailwind CSS
