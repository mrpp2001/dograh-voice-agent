# App Router

Owns page routes, the global provider stack, and the Next.js server route handlers under
`api/`. Reusable UI lives in `../components/` — see `../components/AGENTS.md`.

## Layout and Providers

`layout.tsx` composes the provider stack in a fixed order:

```
ThemeProvider → SentryErrorBoundary → AuthProvider → AppConfigProvider
  → Suspense → OrgConfigProvider → TelephonyConfigWarningsProvider
  → OnboardingProvider → AppLayout
```

The order is load-bearing: `OrgConfigProvider` and everything below it need auth and app
config resolved first. A new global provider goes inside this chain at the depth its
dependencies require, not at the top. Dark is the locked default — an inline pre-hydration
script sets it, and only an explicit stored `light` opts out.

## Server Route Handlers (`api/`)

These are Next handlers, not backend endpoints.

- `api/config/*` — runtime config the browser fetches instead of baking into the bundle
  (auth provider, PostHog, Sentry, version).
- `api/auth/*` — session, logout, and OSS token handling.
- `api/v1/[...path]` — proxy to the backend. Strips hop-by-hop headers; `runtime = "nodejs"`
  and `dynamic = "force-dynamic"` are required, not incidental.

## Middleware

`../middleware.ts` gates OSS (`auth_provider === "local"`) deployments only. Two rules to
preserve when editing:

- The auth-provider probe caches only a **definitive** backend answer. Caching a failure would
  poison a module-scoped cache with no TTL for the worker's lifetime.
- `PUBLIC_PATHS` matches on a path-segment boundary (exact or `/`-delimited), never a bare
  prefix — a bare `startsWith` would let `/embed-admin` bypass auth via the `/embed` entry.

## Route-Local Code

`workflow/[workflowId]/` carries its own `components/`, `contexts/`, `hooks/`, `stores/`, and
`utils/`. That code is deliberately colocated because it is workflow-builder-specific. Promote
to `../components/` only when a second route actually consumes it; don't pre-emptively share.
The same pattern appears in `tools/[toolUuid]/components/` and `reports/components/`.

## Conventions

- Fetching in a page or client component must wait for auth readiness, and must check
  `response.error` from the generated client. Both rules and their rationale are in
  `../../AGENTS.md` — follow them; don't restate them here.
- A new page is a directory with `page.tsx`. Add navigation entries in
  `../components/layout/`, not inline in the page.

## Related Context

- Frontend orientation, generated client, auth: `../../AGENTS.md`
- Shared components: `../components/AGENTS.md`
