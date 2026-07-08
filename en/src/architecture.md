# How it works

## Compilation boundary (what enables the shared crate)

| Crate               | Target              | domain `graphql` feature |
|---------------------|---------------------|--------------------------|
| `rust-gql-domain`   | native **and** wasm32 | optional               |
| `rust-gql-backend`  | native              | **on**                   |
| `rust-gql-frontend` | wasm32              | **off**                  |

`domain` is a sibling folder; backend and frontend depend on it by `path`. An
incompatible change to a `domain` type **breaks the compilation of both Rust
sides** at once. That is the second contract (domain logic) that Node cannot
express.

## Pinned versions (latest stable — crates.io, 2026-07-08)

| Crate | Version | Crate | Version |
|-------|---------|-------|---------|
| async-graphql / -axum | 7.2.1 | leptos | 0.8.20 |
| axum | 0.8.9 | figment | 0.10.19 |
| sqlx | 0.9.0 | serde | 1.0.228 |
| tokio | 1.52.3 | garde | 0.23.0 |
| tower-http | 0.7.0 | thiserror | 2.0.18 |

The toolchain is pinned by `rust-toolchain.toml` (channel `1.95.0`, target
`wasm32-unknown-unknown` where relevant).

## Node → Rust mapping

The full mapping (backend **and** frontend/Leptos/WASM), with links to each
dependency and the reasoning behind each choice, has its own page:
[Dependency mapping](./mapping.md).
