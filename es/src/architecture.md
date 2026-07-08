# Cómo funciona

## Frontera de compilación (habilita el crate compartido)

| Crate               | Target              | feature `graphql` de domain |
|---------------------|---------------------|-----------------------------|
| `rust-gql-domain`   | native **y** wasm32 | opcional                    |
| `rust-gql-backend`  | native              | **on**                      |
| `rust-gql-frontend` | wasm32              | **off**                     |

`domain` es una carpeta hermana; backend y frontend dependen de él por `path`.
Un cambio incompatible en un tipo de `domain` **rompe la compilación de ambos
lados Rust** a la vez. Ese es el segundo contrato (lógica de dominio) que Node no
puede expresar.

## Versiones fijadas (últimas estables — crates.io, 2026-07-08)

| Crate | Versión | Crate | Versión |
|-------|---------|-------|---------|
| async-graphql / -axum | 7.2.1 | leptos | 0.8.20 |
| axum | 0.8.9 | figment | 0.10.19 |
| sqlx | 0.9.0 | serde | 1.0.228 |
| tokio | 1.52.3 | garde | 0.23.0 |
| tower-http | 0.7.0 | thiserror | 2.0.18 |

Toolchain fijada por `rust-toolchain.toml` (canal `1.95.0`, target
`wasm32-unknown-unknown` donde aplica).

## Mapeo Node → Rust

El mapeo completo (backend **y** frontend/Leptos/WASM), con enlaces a cada
dependencia y el porqué de cada elección, tiene su propia página:
[Mapeo de dependencias](./mapping.md).
