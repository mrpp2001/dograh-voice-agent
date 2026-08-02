# Pipecat Runtime

Owns live call execution: building the pipeline, instantiating STT/LLM/TTS services from
org configuration, wiring transports, and running the task to completion. Does NOT own the
agent graph or conversation logic — that is `../workflow/`.

## Entry Points

- `run_pipeline.py` — `run_pipeline_telephony` and `run_pipeline_smallwebrtc`. Everything a
  call needs is assembled here: concurrency accounting, pre-call fetch, engine construction,
  integration runtime sessions, recording, tracing, teardown.
- `pipeline_builder.py` — `build_pipeline` (cascaded STT→LLM→TTS) and `build_realtime_pipeline`
  (speech-to-speech). These are different pipelines with different frame contracts; changing
  one rarely applies to the other.
- `service_factory.py` — maps a configured provider + model to a concrete pipecat service.
  Every new STT/LLM/TTS vendor is added here and in `../configuration/`.
- `worker_runner.py` — runs the pipeline task.
- `transport_setup.py` / `transport_params.py` — WebRTC and telephony transport construction.
- `realtime/` — Dograh subclasses of pipecat's realtime services (user-mute gating, greeting
  trigger, node-transition handling, function-call deferral).

## Contracts

- **The `pipecat/` submodule stays close to upstream.** Dograh-specific behavior belongs in
  this directory — subclass in `realtime/` or wrap in a processor rather than patching the
  submodule.
- **Every call registers and unregisters with observability and concurrency.** Active-call
  registration (`../observability/active_calls.py`) feeds the deploy drain signal and the
  autoscaling metric; a path that returns early without unregistering leaks a phantom call and
  blocks deploys. Teardown must be in `finally`.
- User-configured service URLs pass through `api/utils/url_security.validate_user_configured_service_url`
  before use. Never fetch a URL taken from org config directly.
- Provider credentials and model selection come from `../configuration/`; do not read env vars
  for per-org service settings.
- The engine is handed its dependencies through setters (`set_task`, `set_context`,
  `set_transport_output`, …). Keep that direction: pipeline constructs, engine consumes.

## Anti-patterns

- Don't add conversation or node-transition logic here — it belongs in
  `../workflow/pipecat_engine.py`.
- Don't instantiate telephony provider classes directly; resolve through
  `../telephony/` helpers.
- Don't block the event loop. This process serves many concurrent calls, and loop lag is a
  published autoscaling signal (`/health/active-calls`).

## Related Context

- Graph and engine: `../workflow/AGENTS.md`
- Provider/model configuration: `../configuration/`
- Telephony transports: `../telephony/AGENTS.md`
- Backend rules: `../../AGENTS.md`
