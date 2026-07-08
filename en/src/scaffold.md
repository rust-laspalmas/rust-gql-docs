# What was built

Three sibling folders under `gql/`, plus the untouched Node SPA:

```
gql/
├── rust-gql-domain/     # PURE shared crate (native + wasm32)
├── rust-gql-backend/    # workspace: crates/api (bin) + xtask + xcheck
├── rust-gql-frontend/   # Leptos CSR crate (wasm32)
├── rust-gql-docs/       # this mdBook
└── gofigeeks-gql-frontend/   # Node SPA — UNTOUCHED
```

Everything hinges on a **single `schema.graphql`**, emitted by the backend and
consumed by three clients:

| Consumer | Role |
|----------|------|
| **async-graphql** | the Rust backend — **emits** it |
| **graphql-client** | the Leptos client — compiles against it |
| **graphql-codegen** | the Node SPA — generates types from it |

Each piece has its own page:

- [Shared domain](./domain.md) — newtypes + validation (native **and** wasm)
- [Backend](./backend.md) — schema, data, transport, auth, storage, subscriptions
- [Leptos frontend](./frontend.md) — wasm client with the contract checked at compile time
