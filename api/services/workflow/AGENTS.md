# Workflow

Owns the agent graph: its schema, validation, the node-spec catalog, and the conversation
engine that walks the graph during a call. Does NOT own pipeline/transport construction
(`../pipecat/`) or post-call integration dispatch (`api/tasks/run_integrations.py`).

## Where Things Live

- `dto.py` — ReactFlow DTOs and per-node data models. `_CORE_NODE_DATA_CLASSES` maps node type
  name to data model; `sanitize_workflow_definition` strips UI-only fields before persisting.
- `node_specs/` — the `NodeSpec` registry the UI and SDK render from. Core specs are generated
  lazily from `dto.py`'s `_CORE_NODE_DATA_CLASSES`; integration specs merge in via
  `all_node_specs()`. Served to clients by `api/routes/node_types.py`.
- `workflow_graph.py` — `Node`, `Edge`, `WorkflowGraph`; acyclicity, connection-count and
  node-config validation; `{{ variable }}` template extraction; transition tool naming.
- `pipecat_engine.py` (+ `pipecat_engine_*.py`) — runtime engine: node transitions, LLM context
  composition, summarization, variable extraction, custom tools, call termination.
- `tools/` — tools exposed to the LLM (calculator, timezone, knowledge base, MCP, custom HTTP,
  transfer resolution).
- `qa/` — post-call analysis, metrics, node summaries.
- `text_chat_*.py`, `embed_*.py` — the text/embed channel equivalents of a voice run.
- `run_creation.py`, `configuration_policy.py`, `trigger_paths.py` — run setup and policy.

## Contracts

- **Adding a core node type** means: a data model in `dto.py`, an entry in
  `_CORE_NODE_DATA_CLASSES`, and handling in `pipecat_engine.set_node`. The spec and the UI
  form follow automatically — do not hand-write a spec or a React component for it.
- **A node type belonging to a third party is not a core node.** It goes in an integration
  package (`../integrations/AGENTS.md`) and registers itself; it must not touch `dto.py`.
- `register()` in `node_specs/` rejects duplicate names. One spec per node type, registered at
  module top level.
- Transition tool names are derived from edge labels and must be unique per source node —
  `validate_unique_transition_tool_names` enforces it. Two edges whose labels normalize to the
  same slug is a graph error, not a runtime surprise.
- Graph validation is the gate before execution. Add new structural rules to
  `WorkflowGraph._validate_graph`, not to the engine.

## Anti-patterns

- Don't bypass `sanitize_workflow_definition` when persisting a definition — UI runtime state
  (`invalid`, `validationMessage`) then leaks into stored workflow JSON.
- Don't treat `sanitize_workflow_definition` as validation. It strips unknown fields only; it
  does not enforce required fields, so partial drafts save cleanly by design.
- Don't build pipeline components here. The engine receives its task, context, and transport
  via setters from `../pipecat/`.

## Related Context

- Live pipeline that drives this engine: `../pipecat/AGENTS.md`
- Integration-provided nodes: `../integrations/AGENTS.md`
- Backend rules: `../../AGENTS.md`
