# Backend (`rust-gql-backend`)

A workspace with `resolver = "3"` and three members: **`crates/api`** (lib + bin
`gql-api`), **`xtask`** (contract) and **`xcheck`** (cross-contract). Dependency
versions live in a single `[workspace.dependencies]` (never repeated across
crates).

## Schema and contract

The `api` lib defines the **code-first schema** with async-graphql and exposes
`build_schema()` (used by `xtask` to emit the SDL):

- A `Query { hello }` root.
- The GraphQL projection of `User` — `String` fields from the domain newtypes,
  auto camelCase.
- A custom **`DateTimeISO`** scalar (RFC3339) matching the Node graphql-scalars one.
- `User` and `Role` are registered explicitly (`register_output_type`): the Node
  contract declares them even though no query returns them yet.

## Data and DataLoader

`db` module:

- `UserRepository` with **sqlx** against the existing Postgres.
- Projects the `users` table (only the **7 contract columns**, validated through
  the domain newtypes) to `type User`.
- `find_by_ids` uses `id = ANY($1)` — **one query for N keys**.
- `loader::UserLoader` wraps it in an async-graphql `DataLoader` (**N+1 fix**).

## Transport and CORS

`http` module:

| Route | Serves |
|-------|--------|
| `POST /graphql` | operations (async-graphql-axum) |
| `GET /graphql` | GraphiQL IDE |
| `GET/POST /graphql/sse` | subscriptions over SSE |
| `GET /graphql/ws` | graphql-ws subscriptions |

tower-http CORS with origins and credentials from `[cors]` (session cookies ⇒ **no
wildcard**).

## Session auth

`auth` module: `trait AuthProvider` + `SessionAuthProvider`.

- Validates the `better-auth.session_token` cookie: **optional** HMAC-SHA256
  signature (per `[auth.session].verify_signature`) + a `sessions` lookup.
- Produces a `Principal`, exposed both in the **GraphQL context** and as an axum
  **`Extension`**.
- The cookie contract is **verified against `better-auth@1.6.19`**.

## Storage and uploads

`storage` module: `trait Storage` + `SupabaseStorage`. It backs the
`uploadMedia(file: Upload!)` mutation, which receives files via the **jaydenseric
multipart spec** (apollo-upload-client compatible, parsed by async-graphql-axum)
and uploads them to the `[storage]` bucket.

## Subscriptions

`SubscriptionRoot { count }`, served over **two feature-selected transports**:

- `subs-sse` — an SSE adapter, graphql-sse compatible (the Node SPA consumes it
  without a WebSocket).
- `subs-ws` — async-graphql-axum's native graphql-ws.

## Features

Orthogonal, feature/config-selected:

| Group | Options |
|-------|---------|
| Auth | `auth-session` · `auth-jwt` · `auth-oidc` |
| Subscriptions | `subs-sse` · `subs-ws` |
| Storage | `storage-supabase` · `storage-s3` |

## `xtask` — the contract

- `emit-schema` writes `schema.graphql` from `build_schema().sdl()`.
- `diff` parses both the emitted SDL and the Node `.gql` to an AST and compares a
  canonical model (type→fields, enum→values), ignoring descriptions, the `schema{}`
  block, built-in directives and orphan scalars. It fails on *breaking* changes
  (removing/changing a type or field) or if the committed `schema.graphql` is stale;
  additive changes are informational.
