# Pedal — Bike Rental Management

A mobile-first React + TypeScript bike-rental app with a Supabase PostgreSQL foundation. The public landing/catalog experience, catalog search/category/sort, bike details, auth entry, booking request dialog, and atomic booking/payment/notification creation are included. Administrative and customer account-management pages are **not yet implemented**; see the current scope section below before using this as a production system.

## Local setup

Requirements: Node.js 20+ and npm.

```sh
npm install
cp .env.example .env.local
```

Create a Supabase project, then set these Vite public variables in `.env.local`:

```dotenv
VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_PUBLISHABLE_OR_ANON_KEY
```

The browser must only receive the publishable/anon key. Never put a Supabase service-role key in a `VITE_` variable or in this repository.

Apply `supabase/migrations/202610270001_bike_rental.sql` in the Supabase SQL Editor (or with the Supabase CLI). The migration creates the two-role profile model, tables, indexes, RLS, signup profile trigger, and transactional RPC functions for booking, reservation decisions, pickup/return, payments, and cancellation. New Auth signups are assigned `User` by the database trigger; frontend metadata cannot grant `Admin`.

After signing up and confirming the chosen administrator's email, promote that account from the Supabase SQL Editor using the commented operator-only SQL at the end of the migration. This step requires a trusted project operator. Never make the update available to public users.

Add bicycles in the Supabase SQL Editor for now, for example:

```sql
insert into public.bicycles (name, category, description, hourly_rate, daily_rate)
values ('Coastline Cruiser', 'City Bike', 'Comfortable city ride.', 18, 100);
```

Run the app:

```sh
npm run dev
```

Production bundle check:

```sh
npm run build
```

## Supabase Next.js helpers

This project is Vite + React, not Next.js, so Next's `page.tsx`, `cookies()`, and middleware session-refresh examples are not used by this app. For an existing Next.js app, the provided `utils/supabase/{client,server,middleware}.ts` and root `middleware.ts` use `@supabase/ssr`; set `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` in that app's ignored `.env.local`. The supplied URL/key have also been placed in this workspace's gitignored `.env.local`. The publishable key is intended for client use, but can be rotated in Supabase if it was exposed unintentionally.

## Vercel deployment

Import this repository into Vercel, set `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` under Project Settings → Environment Variables, and deploy. Add the deployed site's URL to Supabase Authentication → URL Configuration (Site URL and allowed redirect URLs). Redeploy after changing build-time Vite variables.

## Current implementation boundaries

The catalog is a visually illustrative sample until a Supabase project is configured; with Supabase configured the app reads active bicycle rows from the database. Reservation submission calls the `create_reservation` database function, which validates availability, prices from database rates, and writes a pending reservation, payment record, and notification together. The confirmation state is not shown as approved until an administrator approves it.

The current UI does not yet include the requested full user dashboard, reservation history/rental tracking screens, complete registration/reset UX, admin console and CRUD interfaces, image upload UI/storage setup, charts/reports, installable PWA icons/service worker, or end-to-end security/workflow test coverage. The SQL exposes some admin workflow RPCs but those operations have not been exercised against a configured Supabase project. Do not treat this starter as a complete production rental system until those interfaces and Supabase integration tests are added.
