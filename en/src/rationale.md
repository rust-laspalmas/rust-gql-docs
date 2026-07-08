# Why

## A contract the compiler verifies on both sides of the wire

GraphQL stops being an "API format" and becomes a contract the compiler checks in
both Rust clients:

- The emitted SDL (`schema.graphql`) feeds graphql-client (Leptos) and
  graphql-codegen (Node SPA). An incompatible change breaks generation.
- The `domain` crate adds a **second** contract — domain rules that run
  identically in the resolver (native) and in the browser (wasm) — that Node
  cannot express. A validation rule is written once and cannot drift between
  server and client because it is the same code compiled to two targets.

## Why 3 folders and not a single workspace

Keeping `domain` as a sibling folder preserves the key property (a schema change
breaks both Rust sides) **without** mixing a `sqlx` + `tokio` backend and a
`wasm` frontend in the same workspace, where features and targets fight each
other.

## Why auth and subscriptions are extension points

Flexibility is modeled as `trait` + `feature`, not as a premature decision:

- **Auth** — `trait AuthProvider` with impls `session` (replicates the
  Better-Auth cookie/session contract the SPA needs), `jwt` and `oidc`,
  selectable by config.
- **Subscriptions** — a shared `SubscriptionRoot` with `sse` (Node SPA compat)
  and native `ws` adapters.

The default set is what the existing Node SPA needs; the rest stays behind a
feature, without building all five implementations at once.

## Why config-driven

The Node backend's scattered literals (port `4000`, `baseURL`, casing) are
consolidated into `config.toml` + environment. Configuration is data, not code:
switching auth strategy or subscription transport is done **without recompiling
logic**, just by editing the TOML or an environment variable.
