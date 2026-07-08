# Verification

Everything documented here is confirmed by execution (not by assumption).

## `rust-gql-domain`

```bash
cd rust-gql-domain
cargo build                                   # native
cargo build --target wasm32-unknown-unknown   # wasm
cargo build --features graphql                # with async-graphql derives
cargo test                                    # 7 tests (Role serde + User/newtype validation)
cargo clippy --all-targets -- -D warnings
```

## `rust-gql-backend`

```bash
cd rust-gql-backend
cargo build --workspace
cargo clippy --workspace --all-targets -- -D warnings

# fail-fast without the secret -> error naming the field
cargo run -p api

# serves GraphQL at http://localhost:4000/graphql (POST) + GraphiQL (GET)
GQL_DATABASE__URL='postgres://user:pw@localhost:5432/gql' RUST_LOG=info cargo run -p api
# in another terminal:
curl -s -X POST localhost:4000/graphql -H 'content-type: application/json' \
  -d '{"query":"{ hello }"}'   # -> {"data":{"hello":"Hello World!"}}

# subscription over SSE (event stream):
curl -sN -X POST localhost:4000/graphql/sse -H 'content-type: application/json' \
  -d '{"query":"subscription { count(to: 3) }"}'   # -> 3x event: next

# xtask: emit the SDL and verify the contract against the Node .gql files
cargo run -p xtask -- emit-schema   # -> schema.graphql
cargo run -p xtask -- diff          # exit 0 if no breaking change and not stale
```

## `rust-gql-frontend`

```bash
cd rust-gql-frontend
cargo build --target wasm32-unknown-unknown   # graphql-client checks HelloQuery vs schema.graphql
cargo test --lib                              # config_parses
cargo clippy --target wasm32-unknown-unknown --all-targets -- -D warnings
# contract binding: edit src/queries/hello.graphql to a non-existent field
# -> the wasm build FAILS ("No field named ... on Query")
```

## `gofigeeks-gql-frontend/codegen` (third consumer)

```bash
cd gofigeeks-gql-frontend/codegen
npm install
npm run generate   # generates generated/types.ts from ../../rust-gql-backend/schema.graphql
# binding: an operation with a non-existent field fails `graphql-codegen`
```

## Run the Leptos client in the browser (no Trunk)

```bash
scripts/build-web.sh            # debug (includes console_error_panic_hook)
scripts/build-web.sh --release  # lean production (hook compiled out)
# the wasm-bindgen CLI must match the crate version (the script validates and aborts otherwise)

cd rust-gql-frontend && python3 -m http.server 5173   # 5173 = CORS-allowed origin
# with the backend on :4000, open http://localhost:5173
# -> "hello: Hello World!" (real fetch) + Email::parse running in wasm
```

## Cross-contract (Leptos ≡ Node)

```bash
scripts/xcheck.sh
# starts the backend, issues the same `hello` operation from the Node client
# (graphql-codegen) and the Leptos request (graphql_client, same hello.graphql)
# -> PASS when the `data` is identical: {"hello":"Hello World!"}
```

## This book

```bash
# book only (per language)
mdbook build rust-gql-docs/en   # -> rust-gql-docs/book/en
mdbook build rust-gql-docs/es   # -> rust-gql-docs/book/es

# full site: book (ES/EN) + API reference (rustdoc) in parallel
scripts/build-docs.sh           # -> book/{es,en} + book/rustdoc
```

## Global gate (all stages)

```bash
cargo clippy --workspace --all-targets -- -D warnings && cargo fmt --all --check
```
