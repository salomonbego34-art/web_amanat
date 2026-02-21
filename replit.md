# PulseNews

## Overview

PulseNews is a community-driven news aggregation platform (similar to Hacker News) where users can submit articles, upvote them, and browse a curated feed. The app features Replit Auth for authentication, a PostgreSQL database for persistence, and a clean green-themed UI.

The stack is a full-stack TypeScript monorepo:
- **Frontend**: React + Vite + TailwindCSS + shadcn/ui
- **Backend**: Express.js REST API
- **Database**: PostgreSQL with Drizzle ORM
- **Auth**: Replit OpenID Connect (OIDC) integration

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Monorepo Structure
The project uses a three-folder monorepo pattern:
- `client/` — React frontend (Vite dev server, builds to `dist/public/`)
- `server/` — Express backend (runs on Node, builds to `dist/index.cjs`)
- `shared/` — Shared types, schemas, and API route contracts used by both client and server

### Frontend Architecture
- **Framework**: React 18 with TypeScript, bundled by Vite
- **Routing**: Uses `wouter` (lightweight client-side router), NOT react-router
- **State/Data Fetching**: TanStack React Query for server state management
- **UI Components**: shadcn/ui (new-york style) with Radix UI primitives, styled with TailwindCSS
- **Forms**: react-hook-form with zod resolvers for validation
- **Path aliases**: `@/` maps to `client/src/`, `@shared/` maps to `shared/`
- **Theming**: Green-focused color palette using CSS custom properties with light/dark mode support
- **Fonts**: Outfit (display) and Plus Jakarta Sans (body)

### Backend Architecture
- **Framework**: Express.js running on a raw `http.Server`
- **Dev mode**: Vite middleware serves the frontend with HMR (via `server/vite.ts`)
- **Production mode**: Static files served from `dist/public/` (via `server/static.ts`)
- **API prefix**: All API routes are under `/api/`
- **Route definitions**: Centralized in `shared/routes.ts` as a typed API contract with Zod schemas for input validation
- **Storage pattern**: Interface-based storage (`IStorage`) with a `DatabaseStorage` implementation in `server/storage.ts`

### Database
- **Engine**: PostgreSQL (required — `DATABASE_URL` env var)
- **ORM**: Drizzle ORM with `drizzle-zod` for schema-to-validation bridging
- **Schema location**: `shared/schema.ts` (articles, upvotes) and `shared/models/auth.ts` (users, sessions)
- **Migrations**: Managed via `drizzle-kit push` (`npm run db:push`)
- **Tables**:
  - `users` — User profiles (id, email, name, role, approval status)
  - `sessions` — Express session storage (connect-pg-simple)
  - `articles` — Submitted articles (title, url, author reference)
  - `upvotes` — Article upvotes (article + user references)
- **Relations**: Articles belong to users (author), upvotes link users to articles

### Authentication
- **Method**: Replit Auth via OpenID Connect (OIDC)
- **Session storage**: PostgreSQL-backed sessions using `connect-pg-simple`
- **Implementation**: Located in `server/replit_integrations/auth/`
- **Client hook**: `useAuth()` at `client/src/hooks/use-auth.ts` fetches `/api/auth/user`
- **Login flow**: Redirect to `/api/login`, callback handled by Passport.js OIDC strategy
- **Logout**: Redirect to `/api/logout`
- **Admin features**: Role-based access (admin/user), user approval system

### Build System
- **Dev**: `tsx server/index.ts` runs the Express server with Vite middleware
- **Build**: Custom `script/build.ts` runs Vite build for client + esbuild for server
- **Output**: `dist/public/` (client assets) and `dist/index.cjs` (server bundle)
- **Server bundling**: Select dependencies are bundled (allowlisted) to reduce cold start times; others are externalized

### Key Pages
- `/` — Landing page (unauthenticated) or Feed (authenticated)
- `/feed` — Article feed with upvoting
- `/submit` — Article submission form (protected route)

## External Dependencies

### Required Services
- **PostgreSQL Database**: Required. Connection via `DATABASE_URL` environment variable. Used for all data storage including sessions.
- **Replit Auth (OIDC)**: Authentication provider. Requires `ISSUER_URL` (defaults to `https://replit.com/oidc`), `REPL_ID`, and `SESSION_SECRET` environment variables.

### Key NPM Packages
- `drizzle-orm` + `drizzle-kit` — Database ORM and migration tooling
- `express` + `express-session` — HTTP server and session management
- `connect-pg-simple` — PostgreSQL session store
- `passport` + `openid-client` — OIDC authentication
- `@tanstack/react-query` — Client-side data fetching/caching
- `zod` + `drizzle-zod` — Schema validation (shared between client and server)
- `wouter` — Client-side routing
- `date-fns` — Date formatting
- `react-hook-form` + `@hookform/resolvers` — Form handling
- `shadcn/ui` ecosystem (Radix UI primitives, tailwindcss, class-variance-authority, clsx, tailwind-merge)