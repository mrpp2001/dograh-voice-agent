# Components

Owns reusable React components. Route-specific components stay colocated under
`../app/<route>/components/` — see `../app/AGENTS.md`.

## Feature Slices

- `ui/` — shadcn/ui primitives. Regenerate or extend via shadcn; don't fork a primitive to
  change one style, use `cn()` and props.
- `flow/` — React Flow workflow builder: nodes, edges, the spec-driven renderer, node/tool/
  document selectors.
- `workflow/`, `workflow-runs/` — reusable workflow and run UI, including the conversation
  view and its adapters.
- `layout/` — `AppLayout`, navigation, shell. New pages register their nav entry here.
- `telephony/`, `billing/`, `auth/`, `onboarding/`, `filters/`, `http/`, `lead-forms/` —
  feature slices.
- Loose `*.tsx` at the top level (`ServiceConfigurationForm`, `VoiceSelector`,
  `AIModelConfigurationV2Editor`, …) are legacy placement. Put new work in a slice directory;
  don't add to the flat list.

## The Node Renderer Contract

`flow/renderer/` renders workflow nodes from the backend `NodeSpec` catalog:
`useNodeSpecs()` fetches `/api/v1/node-types` once per session and caches it in module scope,
`GenericNode` renders any node type from its spec, and `NodeEditForm` + `PropertyInput` build
the edit form from `PropertySpec` entries.

**Adding a workflow node type is a backend change, not a frontend one.** Register the spec in
`api/services/workflow/node_specs/` (or an integration package) and the UI picks it up. Do not
write a bespoke node component or a hand-maintained node list.

When a property needs a renderer the spec system can't express, extend `PropertyInput` and
`propertyRendererOptions.ts` so every node type gains it — don't special-case one node type.

## Conventions

- File uploads: hidden `<input type="file">` triggered by a visible `<Button>`. Never a visible
  `<Input type="file">`.
- API calls follow the auth-readiness and `response.error` rules in `../../AGENTS.md`.
- Import types from `@/client/types.gen` rather than redeclaring backend shapes locally —
  `../client/` is generated and is the contract.

## Anti-patterns

- Don't import from `../app/` into a shared component. The dependency runs one way: routes
  consume components. (`flow/nodes/GenericNode.tsx` currently imports the workflow route's
  `WorkflowContext` — that is existing debt, not a pattern to copy.)
- Don't hand-edit anything under `../client/`.

## Related Context

- Frontend orientation: `../../AGENTS.md`
- Pages and providers: `../app/AGENTS.md`
- Node specs: `api/services/workflow/AGENTS.md`
