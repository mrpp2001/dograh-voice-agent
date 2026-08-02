# Database Layer

Owns SQLAlchemy models and all persistence access. Does NOT own business rules — services
call these clients, clients do not call services.

## Shape

- `models.py` — every ORM model in one module.
- `db_client.py` — `DBClient` composes ~23 per-domain client mixins into one class.
- `__init__.py` — exports the process-wide `db_client` singleton. Import that:
  `from api.db import db_client`. Do not instantiate `DBClient()` yourself.
- `<domain>_client.py` — one mixin per domain (workflow, campaign, telephony config, …).
- `base_client.py` — `BaseDBClient`: engine/session setup plus `execute_raw_query`.
- `filters.py` — shared query filter construction.
- Migrations live in `../alembic/versions/`.

## Contracts

- **New persistence methods go on the domain mixin that owns the table**, not on `DBClient`
  and not in a service. Register any new mixin in `db_client.py`'s base list or it is unreachable.
- **Org-scoped queries filter at the query level.** Pass `organization_id` into the client
  method and apply it in the SQLAlchemy statement — never fetch broadly and filter in Python.
  This is the tenant-isolation boundary described in `../AGENTS.md`.
- **Method names carry the scope.** A lookup that takes an `organization_id` should require it,
  not default it to `None`, so a caller cannot silently opt out of the check.
- `execute_raw_query` is for reporting/aggregation only. Parameterize — never interpolate
  caller input into the SQL string.

## Migrations

```bash
./scripts/makemigrate.sh "description"  # autogenerate
./scripts/migrate.sh                    # apply
```

Review the generated file before committing — autogenerate misses server defaults, enum value
changes, and index renames.

## Pitfalls

- `database.py` defines its own `engine`/`async_session` (with `echo=True`) and **nothing
  imports it**. The live engine is the one built in `base_client.py`. Don't wire new code to
  `database.py` assuming it is the shared session factory.
- `DBClient` is mixin-composed, so a method name collision across two client modules silently
  resolves by MRO. Keep method names domain-prefixed when there is any chance of overlap.

## Related Context

- Backend rules and org scoping: `../AGENTS.md`
- Callers: `../routes/AGENTS.md`, `../services/`
