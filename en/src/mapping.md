# Dependency mapping (Node → Rust)

Each Rust choice replicates the **role** of its Node counterpart. The crate name
links to its crates.io page (its source); the **Why** column summarizes the reason.

## Backend

| Node | Rust | Why |
|------|------|-----|
| GraphQL Yoga | [async-graphql](https://crates.io/crates/async-graphql) + [async-graphql-axum](https://crates.io/crates/async-graphql-axum) | mature code-first, axum integration, better performance |
| Drizzle + Postgres | [sqlx](https://crates.io/crates/sqlx) | same philosophy (explicit SQL, not ActiveRecord) + compile-time checking |
| DataLoader | [async-graphql](https://crates.io/crates/async-graphql) (`dataloader`) | built in, no separate library |
| Better-Auth | custom trait + [jsonwebtoken](https://crates.io/crates/jsonwebtoken) / [argon2](https://crates.io/crates/argon2) | no equivalent; assembled; `session` replicates the real cookie |
| Supabase Storage | [reqwest](https://crates.io/crates/reqwest) (/ [aws-sdk-s3](https://crates.io/crates/aws-sdk-s3)) | S3-compatible REST over HTTP |
| graphql-sse (subs) | async-graphql SSE + graphql-ws | SSE SPA-compat + native WS |
| apollo-upload-client | native jaydenseric multipart | parsed by async-graphql-axum |
| Runtime / HTTP | [tokio](https://crates.io/crates/tokio) + [axum](https://crates.io/crates/axum) + [tower-http](https://crates.io/crates/tower-http) | Rust's async standard |
| Config (.env) | [figment](https://crates.io/crates/figment) | layered TOML + env, typed |
| Validation | [garde](https://crates.io/crates/garde) | wasm-safe, derive-based |
| Errors | [thiserror](https://crates.io/crates/thiserror) + [anyhow](https://crates.io/crates/anyhow) | typed (library) vs context (binary) |
| Cookie signature | [hmac](https://crates.io/crates/hmac) + [sha2](https://crates.io/crates/sha2) + [base64](https://crates.io/crates/base64) + [subtle](https://crates.io/crates/subtle) | reproduce the Better-Auth signature; constant-time compare |

## Frontend

The role React/Vue/Angular plays in the workshop is filled by **Leptos**, and the
Apollo/urql GraphQL layer by **graphql-client**.

| Workshop role (JS) | Rust | Why |
|--------------------|------|-----|
| React / Vue / Angular (SPA) | [leptos](https://crates.io/crates/leptos) (CSR → **WASM**) | *fine-grained* reactivity (signals) ≈ Solid/Vue; components + router |
| Apollo / urql (GraphQL layer) | [graphql-client](https://crates.io/crates/graphql_client) | generates types from the schema and **checks them at compile time** |
| `useQuery` (loading/error/data) | Leptos `LocalResource` | a direct analogue of useQuery |
| Vite / webpack | [wasm-bindgen](https://crates.io/crates/wasm-bindgen) (+ optional Trunk) | same role (wasm bundling); here run **without Trunk** |
| fetch / axios | [gloo-net](https://crates.io/crates/gloo-net) | wrapper over the browser `fetch`/`EventSource` |
| FormData / EventSource | [web-sys](https://crates.io/crates/web-sys) / [js-sys](https://crates.io/crates/js-sys) | browser APIs (multipart upload, SSE) |
| Readable panics (dev) | [console_error_panic_hook](https://crates.io/crates/console_error_panic_hook) | Rust panics in the console (**debug-only**) |

## Why these choices (and not others)

- **async-graphql (not Juniper)** — more actively maintained code-first, axum
  integration, and DataLoader / subscriptions / uploads out of the box.
- **sqlx (not SeaORM/Diesel)** — keeps Drizzle's spirit (explicit SQL) and checks
  SQL against the DB. We use *runtime queries* (not the `query!` macros) so it
  **compiles without a database available**.
- **reqwest with `rustls`** — avoids system OpenSSL → portable (Linux/macOS) and a
  self-contained binary.
- **Leptos (not Yew/Dioxus)** — *fine-grained* reactivity (signals), closer to
  Vue/Solid than React's VDOM; direct CSR → wasm.
- **graphql-client (not cynic)** — the macro reads schema + query and **fails the
  build** if they mismatch; the project's differentiator.
- **gloo-net 0.6 (not 0.7)** — unified with the version Leptos resolves, to avoid
  two copies in the tree.
- **plain wasm-bindgen (no Trunk)** — less tooling; the only care is that the CLI
  version **matches** the crate's (`build-web.sh` validates it).

The exact pinned versions live in [How it works](./architecture.md).
