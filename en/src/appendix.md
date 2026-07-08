# Appendix: GraphQL in Rust

## The two pillars

- **`async-graphql`** (server) — a **code-first** approach (Rust types are the
  schema; the SDL is emitted). It integrates with axum/poem/actix and ships
  DataLoader, subscriptions (WS **and** SSE), multipart uploads, custom scalars,
  directives, Apollo Federation, extensions/tracing, and query complexity/depth
  limits.
- **`graphql-client`** (client) — generates Rust types from the `schema.graphql` +
  the `.graphql` operations and **checks them at compile time**. Transport-agnostic
  (reqwest on native, gloo-net on wasm).

## async-graphql vs Node (Yoga) — capabilities

| Capability | async-graphql (Rust) | Node (Yoga) |
|------------|----------------------|-------------|
| Approach | code-first (or SDL-first) | SDL-first |
| DataLoader | built in | `dataloader` library |
| Subscriptions | native WS + SSE | graphql-sse |
| Uploads | native jaydenseric multipart | apollo-upload-client |
| Custom scalars | `#[Scalar]` | graphql-scalars |
| Federation | Apollo Federation | Mesh / Apollo |
| Runtime type-safety | guaranteed | erased (TS) |
| Client contract | `graphql-client` (compiles) | graphql-codegen (build step) |

## Other ecosystem pieces

- **Servers**: async-graphql (+ axum/poem/actix), Juniper (code-first, older).
- **Client**: graphql-client (codegen), cynic (typed query builder).
- **Data**: sqlx (compile-time-checked SQL), SeaORM, Diesel.
- **WASM / UI**: Leptos, Dioxus, Yew — all pair with graphql-client.

## Examples implemented in this project

Everything below is built and **verified by execution**. The **Code** column links
to the module in the **API reference (rustdoc)**, generated alongside the book:

| Example | Code (rustdoc) | What it shows |
|---------|----------------|---------------|
| `hello` query | [`api::graphql`](../rustdoc/api/graphql/index.html) | minimal code-first schema |
| `User` projection + `DateTimeISO` scalar | [`api::graphql`](../rustdoc/api/graphql/index.html) | domain types → SDL, custom scalar |
| N+1-safe DataLoader | [`api::db`](../rustdoc/api/db/index.html) · [`api::loader`](../rustdoc/api/loader/index.html) | `id = ANY($1)` + `DataLoader` |
| Better-Auth session auth | [`api::auth`](../rustdoc/api/auth/index.html) | HMAC-SHA256 cookie → `Principal` |
| Multipart upload | [`api::storage`](../rustdoc/api/storage/index.html) | `uploadMedia(Upload!)` jaydenseric |
| SSE + WS subscriptions | [`api::graphql`](../rustdoc/api/graphql/index.html) · [`api::http`](../rustdoc/api/http/index.html) | `count` over two transports |
| Contract emit + diff | [`xtask`](../rustdoc/xtask/index.html) | `schema.graphql` + breaking/additive gate |
| Typed Leptos client | [Leptos frontend](./frontend.md) chapter | `graphql-client` from the schema |
| Cross-contract | [`xcheck`](../rustdoc/xcheck/index.html) | Leptos ≡ Node → identical result |

The examples are described in the [Backend](./backend.md), [Frontend](./frontend.md)
and [Verification](./verification.md) chapters; this table is the index, with a
direct link to the **code** in the [API reference (rustdoc)](../rustdoc/api/index.html).

> The rustdoc reference is generated together with the book by `scripts/build-docs.sh`
> (mdBook + `cargo doc` in parallel). With a plain `mdbook build`, the `../rustdoc/…`
> links do not exist yet.

## Implemented tests and how to use them

The philosophy: **pure logic tested without external services**; anything touching
the DB or storage sits behind `#[ignore]`; and end-to-end is checked via `curl` /
script.

| Crate | Covers | Notes |
|-------|--------|-------|
| `rust-gql-domain` | `Role` serde; `Email`/`UserId`/`UserName` validation; `User` aggregate | run on native **and** wasm |
| `api` (graphql) | `hello` resolves; **SDL parity** with `user.gql`; `count` stream | — |
| `api` (http) | POST `hello`, GET GraphiQL, **multipart upload** (jaydenseric), **SSE** stream | via `tower::oneshot`, no port or DB |
| `api` (auth) | **real Better-Auth HMAC vector**; cookie parsing | cross-language test |
| `api` (db) | `users`→`User` projection; invalid-row rejection | + `#[ignore]` against a real DB |
| `xtask` | contract diff: identical / removed field / changed type / additive / enum | 3 **negative** cases |
| `rust-gql-frontend` | `config_parses` | contract binding: a non-existent field **won't compile** |

How to run them:

```bash
# per crate
cargo test                        # domain / frontend
cargo test --workspace            # backend (api + xtask + xcheck)

# real integration (needs credentials)
GQL_DATABASE__URL=... cargo test -p api -- --ignored          # session / repo
GQL_STORAGE__URL=... GQL_STORAGE__KEY=... cargo test -p api -- --ignored

# cross-contract (Leptos ≡ Node) against the live backend
scripts/xcheck.sh

# negative binding (must FAIL):
#   edit rust-gql-frontend/src/queries/hello.graphql to a non-existent field
#   -> the wasm build fails ("No field named ... on Query")
```

## Benchmarking suggestions

There are **no benchmarks** in the repo yet; these are the recommended paths:

- **Micro (Rust)** — [criterion](https://crates.io/crates/criterion) for pure
  functions: cookie HMAC verification, `users`→`User` projection, `build_schema()`.
- **GraphQL throughput/latency** — load `POST /graphql` with a fixed query using
  `oha`, `wrk`, `k6` or `vegeta`; compare **Node vs Rust** on p50/p99, RPS and RSS.
- **DB** — [`#[sqlx::test]`](https://docs.rs/sqlx) with an ephemeral test DB to
  measure real queries and the `DataLoader` effect (N+1 with and without).
- **Resolvers** — async-graphql *extensions* (apollo-tracing) give per-resolver
  timings.
- **What to measure** — cold start / binary size, RSS under load, p99, and N+1
  with/without the DataLoader.

> Benchmarking honesty: same DB, warm caches, realistic queries and percentiles (not
> averages). Don't cherry-pick the favorable case.

