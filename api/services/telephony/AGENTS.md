# Telephony

Owns cross-provider telephony: the provider registry, config resolution, transfers, call
status, and media-WebSocket auth. Provider-specific transports, serializers, config models,
and webhook handlers live in `providers/` — **read `providers/AGENTS.md` before changing a
provider package.**

## Shared Surface

- `registry.py` — `ProviderSpec` (capabilities plus the `ProviderUIMetadata` the frontend
  renders the config form from) and `register()`.
- `factory.py` — the only sanctioned way to obtain a provider: by config id, for a workflow
  run, for an organization's default, or matched for an inbound call. Also normalizes
  org-scoped config and loads transport credentials.
- `base.py` — the `TelephonyProvider` contract providers implement.
- `call_transfer_manager.py`, `transfer_event_protocol.py`, `external_pbx.py` — transfer flow.
- `status_processor.py` — normalizes provider call-status callbacks.
- `ws_auth.py` — capability-token auth for the media WebSocket.
- `ari_manager.py` — shared Asterisk/ARI control plane.

## Contracts

- **Registration is import-driven.** This package's `__init__.py` eagerly imports `providers/`
  so every provider self-registers before any consumer runs. `factory.py` and `registry.py`
  must therefore stay free of provider imports — adding one reintroduces the import cycle the
  eager load exists to avoid.
- **Resolve providers through `factory.py`.** Never instantiate a provider class from a route,
  task, or unrelated service.
- **Config lookups stay tenant-safe** and must honour a run-scoped telephony configuration when
  the workflow run carries one — an organization default must not silently override it.
- **Media-WebSocket tokens are secrets.** They travel in the URL path, not the query string,
  and stay redacted in logs.
- Shared route glue lives in `api/routes/telephony.py`, which auto-mounts provider routers.
  Provider HTTP handlers belong in `providers/<name>/routes.py`; not every provider has one
  (`ari` is transport-only and is skipped by the auto-mounter).

## Anti-patterns

- Don't add an `if provider == "twilio"` branch here. Cross-provider code branches on
  `ProviderSpec` capabilities; anything else belongs in the provider package.
- Don't add provider config fields to `api/schemas/` — they live in the provider package's own
  config model and reach the UI through `ProviderUIMetadata`.
- `README.md` in this directory is stale: it describes flat `*_provider.py` files, schema edits
  in `api/schemas/telephony_config.py`, and direct `TwilioService` usage. Trust the code and
  these docs over it.

## Related Context

- Provider packages: `providers/AGENTS.md`
- Pipeline consuming telephony transports: `../pipecat/AGENTS.md`
- Route glue: `../../routes/AGENTS.md`
- Backend rules and org scoping: `../../AGENTS.md`
