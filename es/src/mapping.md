# Mapeo de dependencias (Node → Rust)

Cada elección Rust replica el **rol** de su equivalente Node. El nombre del crate
enlaza a su ficha en crates.io (su fuente); la columna **Por qué** resume la razón.

## Backend

| Node | Rust | Por qué |
|------|------|---------|
| GraphQL Yoga | [async-graphql](https://crates.io/crates/async-graphql) + [async-graphql-axum](https://crates.io/crates/async-graphql-axum) | code-first maduro, integra con axum, mejor rendimiento |
| Drizzle + Postgres | [sqlx](https://crates.io/crates/sqlx) | misma filosofía (SQL explícito, no ActiveRecord) + verificación en compilación |
| DataLoader | [async-graphql](https://crates.io/crates/async-graphql) (`dataloader`) | integrado, sin librería aparte |
| Better-Auth | trait propio + [jsonwebtoken](https://crates.io/crates/jsonwebtoken) / [argon2](https://crates.io/crates/argon2) | no hay equivalente; se ensambla; `session` replica la cookie real |
| Supabase Storage | [reqwest](https://crates.io/crates/reqwest) (/ [aws-sdk-s3](https://crates.io/crates/aws-sdk-s3)) | REST S3-compatible por HTTP |
| graphql-sse (subs) | async-graphql SSE + graphql-ws | SSE compat SPA + WS nativo |
| apollo-upload-client | multipart jaydenseric nativo | lo parsea async-graphql-axum |
| Runtime / HTTP | [tokio](https://crates.io/crates/tokio) + [axum](https://crates.io/crates/axum) + [tower-http](https://crates.io/crates/tower-http) | el estándar async de Rust |
| Config (.env) | [figment](https://crates.io/crates/figment) | capas TOML + env, tipado |
| Validación | [garde](https://crates.io/crates/garde) | wasm-safe, por derive |
| Errores | [thiserror](https://crates.io/crates/thiserror) + [anyhow](https://crates.io/crates/anyhow) | tipados (librería) vs contexto (binario) |
| Firma de cookie | [hmac](https://crates.io/crates/hmac) + [sha2](https://crates.io/crates/sha2) + [base64](https://crates.io/crates/base64) + [subtle](https://crates.io/crates/subtle) | reproducir la firma Better-Auth; comparación en tiempo constante |

## Frontend

El rol que ocupa React/Vue/Angular en el taller lo cubre **Leptos**, y la capa
GraphQL de Apollo/urql la cubre **graphql-client**.

| Rol en el taller (JS) | Rust | Por qué |
|-----------------------|------|---------|
| React / Vue / Angular (SPA) | [leptos](https://crates.io/crates/leptos) (CSR → **WASM**) | reactividad *fine-grained* (signals) ≈ Solid/Vue; componentes + router |
| Apollo / urql (capa GraphQL) | [graphql-client](https://crates.io/crates/graphql_client) | genera tipos del schema y **valida en compilación** |
| `useQuery` (loading/error/data) | `LocalResource` de Leptos | análogo directo de useQuery |
| Vite / webpack | [wasm-bindgen](https://crates.io/crates/wasm-bindgen) (+ Trunk opcional) | mismo rol (bundle wasm); aquí montado **sin Trunk** |
| fetch / axios | [gloo-net](https://crates.io/crates/gloo-net) | wrapper de `fetch`/`EventSource` del navegador |
| FormData / EventSource | [web-sys](https://crates.io/crates/web-sys) / [js-sys](https://crates.io/crates/js-sys) | APIs del navegador (upload multipart, SSE) |
| Panics legibles (dev) | [console_error_panic_hook](https://crates.io/crates/console_error_panic_hook) | panics de Rust en la consola (**debug-only**) |

## Por qué estas opciones (y no otras)

- **async-graphql (no Juniper)** — code-first más activo, integración axum, y
  DataLoader / subscriptions / uploads de fábrica.
- **sqlx (no SeaORM/Diesel)** — mantiene el espíritu de Drizzle (SQL explícito) y
  valida SQL contra la BD. Usamos *runtime queries* (no las macros `query!`) para
  **compilar sin una BD disponible**.
- **reqwest con `rustls`** — evita OpenSSL del sistema → portable (Linux/macOS) y
  binario autocontenido.
- **Leptos (no Yew/Dioxus)** — reactividad *fine-grained* (signals), más cercana a
  Vue/Solid que al VDOM de React; CSR → wasm directo.
- **graphql-client (no cynic)** — el macro lee schema + query y **falla la
  compilación** si no cuadran; es el diferenciador del proyecto.
- **gloo-net 0.6 (no 0.7)** — unificado con la versión que resuelve Leptos, para no
  arrastrar dos copias en el árbol.
- **wasm-bindgen directo (sin Trunk)** — menos tooling; el único cuidado es que la
  versión del CLI **coincida** con la del crate (el `build-web.sh` lo valida).

Las versiones exactas fijadas están en [Cómo funciona](./architecture.md).
