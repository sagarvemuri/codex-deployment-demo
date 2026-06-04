# AGENTS.md - Studio Package Context

## Scope
These instructions apply to `apps/studio`, the Next.js/React/TypeScript
dashboard for hosted and self-hosted Supabase projects. Follow the root
`AGENTS.md` and `/standards` first; this file adds Studio-specific working
context.

## Architecture to preserve
- Studio is primarily a Next Pages Router app. Keep page files thin: pick the
  layout, do minimal route/permission gating, and delegate product behavior to
  `components/interfaces/<feature>`.
- Use layout components for shell and navigation behavior:
  `DefaultLayout` for the dashboard shell, feature layouts for product
  navigation, and `ProjectLayout` for project lifecycle/connection blocking.
- Put reusable UI in `components/ui` or shared workspace packages (`ui`,
  `ui-patterns`). Put tightly coupled feature UI in
  `components/interfaces/<feature>`. Put page structure in
  `components/layouts`.
- Keep business logic out of route files. Prefer `data/*`, `state/*`, `lib/*`,
  hooks, or feature-local `*.utils.ts` files depending on ownership.

## Data fetching
- Use the single OpenAPI fetch boundary in `data/fetchers.ts` for platform and
  v1 API calls. Do not create new ad hoc fetch clients for Studio API data.
- Follow existing data-module structure:
  - `keys.ts` exports explicit query-key factories.
  - `*-query.ts` exports the raw async fetcher and the React Query hook.
  - `*-mutation.ts` exports the raw mutation function and the React Query
    mutation hook.
- Query keys must include every value that changes returned data. Keep
  execution-only flags out of keys when they do not affect returned data.
- Mutations should invalidate the narrow affected query keys. Use broad
  invalidation only when the code cannot know the affected resources, such as
  contextual invalidation after raw SQL editor mutations.
- Let `handleError` normalize API errors. Avoid silent fallbacks; surface errors
  with context or use the feature's established toast/error pattern.

## SQL and database operations
- Route database SQL through `data/sql/execute-sql-query.ts` unless an existing
  specialized boundary applies.
- Prefer `@supabase/pg-meta` helpers and query builders for metadata, DDL, and
  row operations. Do not hand-build SQL strings when a pg-meta helper or safe SQL
  fragment exists.
- Preserve pg-meta connection behavior: hosted calls require the encrypted
  connection string header; self-hosted calls may execute through local API
  routes without a browser-visible connection string.
- If role impersonation is relevant, wrap SQL with the helpers in
  `lib/role-impersonation.ts` and pass the impersonation state through the data
  layer explicitly.
- For analytics log SQL, use the established safe analytics SQL boundary in
  `data/logs/execute-analytics-sql.ts` and branded `SafeLogSqlFragment` flow.

## State model
- Treat React Query as the owner of server/cache state.
- Treat Valtio stores in `state/*` as the owner of interactive client state:
  tabs, table editor panels, grid state, role impersonation, selected database,
  assistant state, and global UI flags.
- Keep project-scoped state under `ProjectContextProvider` so it resets when the
  project ref changes.
- Use URL state and local/session storage only where Studio already treats them
  as product behavior, such as table filters/sorts, schema selection, editor
  tabs, and dashboard history.
- Use component `useState` for local UI-only state. Do not introduce a global
  store for state that belongs to one component tree.

## Hosted and self-hosted behavior
- Always check whether a flow behaves differently under `IS_PLATFORM`.
- Hosted mode talks to Supabase Platform APIs and enforces auth, MFA,
  permissions, and feature flags.
- Self-hosted/CLI mode uses local Next API routes under `pages/api/platform/*`
  to emulate platform-shaped responses and proxy to local Supabase services.
- When adding a platform-facing API call that Studio also needs self-hosted,
  add or update the corresponding local API route instead of special-casing the
  UI.
- Do not expose service keys or raw database connection strings to browser code.

## Permissions, routing, and feature flags
- Use `withAuth`, `RouteValidationWrapper`, `useSelectedProjectQuery`, and
  `useSelectedOrganizationQuery` instead of duplicating route/auth logic.
- Use `useAsyncCheckPermissions` or existing permission helpers for gated UI.
  Self-hosted should continue to bypass platform permissions.
- Use `useIsFeatureEnabled` for feature gates so profile disabled-features and
  local overrides are both respected.

## Verification
- For data/query/mutation changes, run focused Vitest tests where they exist and
  add tests for new logic-heavy utilities.
- For UI workflow changes, verify the relevant route in Studio when practical.
- For SQL generation changes, test the generated SQL or the utility that builds
  it. Prefer isolated tests over relying on visual inspection.
