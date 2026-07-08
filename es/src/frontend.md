# Frontend Leptos (`rust-gql-frontend`)

Crate Leptos CSR (`crate-type = ["cdylib", "rlib"]`) que compila a `wasm32`.

## Contrato verificado en compilación

`graphql-client` genera `HelloQuery` desde `../rust-gql-backend/schema.graphql`
**en tiempo de compilación**: un cambio incompatible del schema **rompe este build
wasm igual que el del backend** — el contrato queda verificado por el compilador a
ambos lados.

## Cómo funciona

- `fetch_hello` ejecuta la query con **gloo-net**.
- El `App` usa `LocalResource` (≈ `create_resource` / useQuery).
- Reusa `domain` en el navegador (`Email::parse`) — la **misma validación** que el
  resolver nativo.
- El endpoint sale de `config.toml` **embebido en compilación** (`include_str!` +
  toml).

## Demos interactivos

El cliente ejercita cada capacidad del backend: validación de dominio reactiva,
query `hello`, subscription `count` por SSE, upload por `uploadMedia`, y un enlace a
GraphiQL. Se ejecuta en el navegador **sin Trunk**, con wasm-bindgen directo (ver
[Verificación](./verification.md)).
