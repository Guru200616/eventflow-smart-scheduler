# EventFlow

A portfolio-grade full-stack event scheduler with JWT auth, CRUD events, calendar view, admin panel, and event sharing — built on React + Vite, Express 5, PostgreSQL + Drizzle ORM.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied to `/api`)
- `pnpm --filter @workspace/eventflow run dev` — run the React frontend (port auto-assigned, proxied to `/`)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — JWT signing secret

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React 19 + Vite + TailwindCSS v4 + shadcn/ui + Recharts + Sonner
- API: Express 5 + Zod validation
- DB: PostgreSQL + Drizzle ORM
- Auth: JWT (bcryptjs hashing, Authorization Bearer tokens)
- API codegen: Orval (from OpenAPI spec → React Query hooks + Zod schemas)
- Router: Wouter

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI source of truth for all endpoints
- `lib/api-client-react/src/` — generated hooks + custom fetch with auth token getter
- `lib/db/src/schema/` — Drizzle schema: `users.ts`, `events.ts`
- `artifacts/api-server/src/routes/` — Express route handlers: auth, events, stats, admin
- `artifacts/api-server/src/middlewares/auth.ts` — JWT auth middleware
- `artifacts/eventflow/src/pages/` — React pages: login, dashboard, events, calendar, admin, settings
- `artifacts/eventflow/src/hooks/use-auth.tsx` — AuthContext + setAuthTokenGetter wiring
- `artifacts/eventflow/src/components/layout.tsx` — Sidebar nav layout

## Architecture decisions

- **OpenAPI-first**: All API contracts defined in `openapi.yaml`; hooks and Zod schemas are generated via Orval — never hand-written.
- **JWT auth via custom fetch**: `setAuthTokenGetter(() => localStorage.getItem("ef_token"))` is called once in `AuthProvider` so all generated API calls automatically include the token.
- **Dark-by-default theme**: CSS variables set in `:root` — no light mode toggle needed for this portfolio project.
- **Path-based routing**: API server mounted at `/api`, frontend at `/`; the shared proxy handles routing without any Vite proxy config.
- **Drizzle enums**: `user_role`, `event_category`, `event_priority`, `event_status`, `event_recurrence` defined in schema and reflected in the OpenAPI spec.

## Product

- **Auth**: Login + register with JWT, persisted to localStorage, admin role support
- **Events**: Full CRUD with search + multi-filter (category, priority, status), recurring events, location
- **Event Sharing**: Share any event with another user by email; shared events appear in their list
- **Calendar**: Monthly grid view with day-level event dots and click-to-expand detail panel
- **Dashboard**: Live stats (total, upcoming, completed, shared), pie charts by category/priority, recent activity feed
- **Admin Panel**: All-users table with promote/demote/delete, all-events table, platform-wide stats

## Demo Credentials

- Admin: `admin@eventflow.com` / `admin123`
- User: `jane@eventflow.com` / `user123`

## Gotchas

- Always run `pnpm --filter @workspace/api-spec run codegen` after editing `openapi.yaml` — generated files are not auto-refreshed.
- Run `pnpm --filter @workspace/db run push` after editing DB schema files.
- Do not call `pnpm dev` at the workspace root — use the workflows instead.
- The frontend uses `BASE_URL` from Vite env for the router base; the API client uses relative URLs via the shared proxy.
