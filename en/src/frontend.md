# Leptos frontend (`rust-gql-frontend`)

A Leptos CSR crate (`crate-type = ["cdylib", "rlib"]`) compiling to `wasm32`.

## Contract checked at compile time

`graphql-client` generates `HelloQuery` from `../rust-gql-backend/schema.graphql`
**at compile time**: an incompatible schema change **breaks this wasm build just as
it breaks the backend's** — the contract is checked by the compiler on both sides.

## How it works

- `fetch_hello` runs the query with **gloo-net**.
- The `App` uses `LocalResource` (≈ `create_resource` / useQuery).
- It reuses `domain` in the browser (`Email::parse`) — the **same validation** as
  the native resolver.
- The endpoint comes from a **compile-time-embedded** `config.toml` (`include_str!`
  + toml).

## Interactive demos

The client exercises each backend capability: reactive domain validation, the
`hello` query, the `count` subscription over SSE, an upload via `uploadMedia`, and a
link to GraphiQL. It runs in the browser **without Trunk**, with plain wasm-bindgen
(see [Verification](./verification.md)).
