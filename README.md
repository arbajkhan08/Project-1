# Junior Papers

A site for students to search and download previous year question papers,
with an auto-scraper feeding new papers in and a monetization layer (ads +
premium unlock) built in from the start.

## Stack
- **Next.js 14 (App Router) + TypeScript** — frontend + API routes
- **Prisma + PostgreSQL** (Neon or Supabase) — database
- **Cloudflare R2** — PDF file storage (S3-compatible, no egress fees)
- **Playwright + Cheerio** — scraper worker
- **Tailwind CSS** — styling

## Getting started

```bash
npm install
cp .env.example .env      # fill in your real DB / R2 credentials
npx prisma migrate dev --name init
npm run dev
```

Visit http://localhost:3000

## Project structure

```
app/
  page.tsx              # homepage with search
  papers/page.tsx        # browse/search results
  api/papers/route.ts    # GET /api/papers (search/filter)
  api/papers/[id]/download/route.ts  # download + counts + future ad-gate
prisma/
  schema.prisma          # College -> Course -> Subject -> Paper data model
lib/
  prisma.ts              # Prisma client singleton
  storage.ts              # R2 upload/signed-URL helpers
scraper/
  scrape.ts              # sample scraper — READ THE COMMENTS before running
```

## Before you point the scraper at a real college site
1. Check the site's `/robots.txt` and respect disallowed paths.
2. Rate-limit requests (already has a 500ms delay between file downloads — raise it if needed).
3. Every scraped paper keeps a `sourceUrl` for provenance — never remove this.
4. New scraped papers are inserted with `approved: false` — review them in
   `/admin` before they go live. This catches scraper mistakes before students see them.
5. Consider emailing the exam cell for informal permission — avoids IP bans later.

## Monetization checklist (in build order)
- [ ] Get real traffic + add a privacy policy page → apply for **Google AdSense**
- [ ] Ad slots are already stubbed in `app/page.tsx` and `app/papers/page.tsx` — swap the placeholder `div`s for the real AdSense `<ins>` snippet once approved
- [ ] Add a `plan` check in the download route: FREE users see a 10s "ad/wait" screen, PREMIUM users skip it
- [ ] Add Razorpay/Stripe for premium subscriptions once you have >500 daily users

## Deployment
- **Vercel** for the Next.js app (`vercel deploy`) — connects directly to this repo
- **Neon** or **Supabase** for Postgres — copy the connection string into `DATABASE_URL`
- **Cloudflare R2** — create a bucket, generate API tokens, fill in `.env`
- Scraper: run via **Vercel Cron** hitting a protected `/api/admin/scrape` route,
  or a small always-on VPS if Playwright's browser binary is too heavy for serverless

## Not yet built (next steps)
- Admin dashboard (`app/admin`) to approve scraped papers and manually upload
- Auth (NextAuth) for student accounts + premium plan
- Full-text search (Postgres `tsvector` first; Meilisearch/Algolia later)
- Ad-gate / unlock flow in the download route
