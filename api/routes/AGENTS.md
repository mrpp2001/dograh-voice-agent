# Routes

Owns the HTTP and WebSocket surface: request parsing, auth resolution, response shaping.
Does NOT own domain logic — that lives in `api/services/` (see `api/AGENTS.md` for the
routes-vs-services rule before adding anything here).

## Aggregation

- Every router is imported and mounted in `main.py`. A new route module is invisible until
  it is added there.
- `main.py` also mounts integration routers via `all_routers()` from
  `api.services.integrations` — integration packages must not be added to `main.py` by hand.
- `api/app.py` mounts the aggregate under `API_PREFIX` (`/api/v1`). Declare the router's own
  path segment as `APIRouter(prefix="/campaign")`; never repeat `/api/v1` locally.
- Telephony provider routers are auto-mounted by `telephony.py`, not by `main.py`. Provider
  HTTP handlers belong in `api/services/telephony/providers/<name>/routes.py`.

## Auth Dependencies

Pick from `api/services/auth/depends.py`; do not roll a local variant.

- `get_user` — the default for authenticated endpoints.
- `get_user_with_selected_organization` — use when the handler needs
  `selected_organization_id` (org-scoped reads/writes).
- `get_user_ws` — WebSocket endpoints.
- `get_superuser` / `require_local_auth` — admin and OSS-local-only surfaces.

Routers under `public/*` are deliberately unauthenticated. They must derive and validate an
`organization_id` from the payload or token themselves — an unauthenticated route is not an
excuse to skip the tenant check.

## Ops Endpoints

`main.py` hosts `/health` plus two operational endpoints guarded by the
`X-Dograh-Devops-Secret` header:

- `/health/active-calls` — per-worker drain signal; deploys wait for zero before SIGTERM.
- `/health/autoscale-metric` — fleet-wide KEDA signal. It returns **503 on Redis failure, not
  0** — a successful `0` tells the HPA to scale down, a 503 makes it hold. Preserve that.

## Anti-patterns

- Don't put orchestration, external API calls, or business rules in a handler. If `tasks/`,
  `mcp_server/`, or another route could reuse it, it belongs in `services/`.
- Don't query models directly — go through the `db_client` clients (`api/db/AGENTS.md`).
- Don't trust an id from the request body to imply ownership; refetch with the caller's
  `organization_id`.
- Don't add a catch-all route module. Extend the domain router that owns the concern.

## Related Context

- Backend rules and org scoping: `../AGENTS.md`
- Data access: `../db/AGENTS.md`
- Telephony routes: `../services/telephony/AGENTS.md`
- Integration routers: `../services/integrations/AGENTS.md`
- Node-type catalog served from `node_types.py`: `../services/workflow/AGENTS.md`
