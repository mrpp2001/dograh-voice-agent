# Dograh - Project Overview

Dograh is a voice AI platform for building and deploying conversational AI agents with telephony and WebRTC support.

## Project Structure

```
dograh/
├── api/              # Backend - FastAPI application
├── ui/               # Frontend - Next.js application
├── scripts/          # Helper scripts for local development
├── docs/             # Mintlify documentation
├── pipecat/          # Pipecat framework (git submodule)
├── docker-compose.yaml       # Production/OSS deployment
├── docker-compose-local.yaml # Local development services
```

## Intent Layer

**Before changing code in a subdirectory, read every `AGENTS.md` on the way down to it.** Parent docs stay navigational; the deepest node owns the local contracts. Don't restate a child's rules in a parent.

| Scope | Node |
| --- | --- |
| Backend orientation, org scoping, routes-vs-services | `api/AGENTS.md` |
| HTTP/WebSocket surface and router aggregation | `api/routes/AGENTS.md` |
| Data access clients and models | `api/db/AGENTS.md` |
| Workflow graph, node specs, conversation engine | `api/services/workflow/AGENTS.md` |
| Live call pipeline and STT/LLM/TTS wiring | `api/services/pipecat/AGENTS.md` |
| Telephony shared contracts | `api/services/telephony/AGENTS.md` |
| Telephony provider packages | `api/services/telephony/providers/AGENTS.md` |
| Integration packages | `api/services/integrations/AGENTS.md` |
| Frontend orientation, generated client, auth | `ui/AGENTS.md` |
| App Router pages and Next route handlers | `ui/src/app/AGENTS.md` |
| Shared React components | `ui/src/components/AGENTS.md` |
| Local dev and ops scripts | `scripts/AGENTS.md` |
| Mintlify documentation | `docs/AGENTS.md` |

Not yet covered by a node: `evals/`, `sdk/`, `deploy/`, `examples/`, `api/mcp_server/`, `api/tests/`. Read the code there; don't assume a parent doc describes it.

To audit or extend this hierarchy, use the repo's own skill at `.agents/skills/review-agents-md/`.

### Global Invariants

- **`AGENTS.md` is the single source of truth for agent context.** Every `CLAUDE.md` here is a one-line `@AGENTS.md` import — keep it that way and never duplicate content into it.
- **Tenant isolation**: every org-scoped read, write, and foreign-key reference filters or validates by `organization_id`. Full rule in `api/AGENTS.md`.
- **`pipecat/` is a git submodule** kept close to upstream. Dograh-specific behavior belongs in `api/services/pipecat/`, never in the submodule.
- **`ui/src/client/` is generated** from the backend OpenAPI spec. Never hand-edit; regenerate with `npm run generate-client`.

## Tech Stack

- **Backend**: Python with FastAPI
- **Frontend**: Next.js 15 with React 19, TypeScript, Tailwind CSS
- **Database**: PostgreSQL with SQLAlchemy (async)
- **Cache/Queue**: Redis with ARQ for background tasks
- **Storage**: MinIO (S3-compatible) for audio files

## Local Development

Contributor setup and service startup are documented in `docs/contribution/setup.mdx`.

## Environment Configuration

- `api/.env` - Backend environment variables. Source this when running repo-owned backend scripts against the dev DB (e.g. `python -m scripts.dump_docs_openapi`).
- `api/.env.test` - Test-only environment variables. Source this when running pytest so tests hit the test DB and never the dev/prod credentials in `api/.env`.
- `ui/.env` - Frontend environment variables

Typical invocation:

```bash
# Tests
source venv/bin/activate && set -a && source api/.env.test && set +a && python -m pytest api/tests/...

# Backend scripts
source venv/bin/activate && set -a && source api/.env && set +a && python -m scripts.dump_docs_openapi
```
