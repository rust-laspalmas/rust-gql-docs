# Apéndice: GraphQL en Rust

## Los dos pilares

- **`async-graphql`** (servidor) — enfoque **code-first** (los tipos Rust son el
  schema; el SDL se emite). Integra con axum/poem/actix; trae DataLoader,
  subscriptions (WS **y** SSE), uploads multipart, scalars custom, directivas,
  Apollo Federation, extensions/tracing, y límites de complejidad/profundidad de
  query.
- **`graphql-client`** (cliente) — genera tipos Rust desde el `schema.graphql` + las
  operaciones `.graphql` y **valida en compilación**. Agnóstico de transporte (usa
  reqwest en native, gloo-net en wasm).

## async-graphql vs Node (Yoga) — capacidades

| Capacidad | async-graphql (Rust) | Node (Yoga) |
|-----------|----------------------|-------------|
| Enfoque | code-first (o SDL-first) | SDL-first |
| DataLoader | integrado | librería `dataloader` |
| Subscriptions | WS + SSE nativos | graphql-sse |
| Uploads | multipart jaydenseric nativo | apollo-upload-client |
| Scalars custom | `#[Scalar]` | graphql-scalars |
| Federation | Apollo Federation | Mesh / Apollo |
| Type-safety en runtime | garantizada | borrada (TS) |
| Contrato cliente | `graphql-client` (compila) | graphql-codegen (paso de build) |

## Otras piezas del ecosistema

- **Servidores**: async-graphql (+ axum/poem/actix), Juniper (code-first, más veterano).
- **Cliente**: graphql-client (codegen), cynic (query builder tipado).
- **Datos**: sqlx (SQL verificado en compilación), SeaORM, Diesel.
- **WASM / UI**: Leptos, Dioxus, Yew — todos combinables con graphql-client.

## Ejemplos implementados en este proyecto

Todo lo de abajo está montado y **verificado por ejecución**. La columna **Código**
enlaza al módulo en la **API doc (rustdoc)**, generada en paralelo al libro:

| Ejemplo | Código (rustdoc) | Qué demuestra |
|---------|------------------|---------------|
| Query `hello` | [`api::graphql`](../rustdoc/api/graphql/index.html) | schema code-first mínimo |
| Proyección `User` + scalar `DateTimeISO` | [`api::graphql`](../rustdoc/api/graphql/index.html) | tipos de dominio → SDL, scalar custom |
| DataLoader anti-N+1 | [`api::db`](../rustdoc/api/db/index.html) · [`api::loader`](../rustdoc/api/loader/index.html) | `id = ANY($1)` + `DataLoader` |
| Auth de sesión Better-Auth | [`api::auth`](../rustdoc/api/auth/index.html) | cookie HMAC-SHA256 → `Principal` |
| Upload multipart | [`api::storage`](../rustdoc/api/storage/index.html) | `uploadMedia(Upload!)` jaydenseric |
| Subscriptions SSE + WS | [`api::graphql`](../rustdoc/api/graphql/index.html) · [`api::http`](../rustdoc/api/http/index.html) | `count` por dos transportes |
| Emisión + diff de contrato | [`xtask`](../rustdoc/xtask/index.html) | `schema.graphql` + gate breaking/aditivo |
| Cliente Leptos tipado | capítulo [Frontend Leptos](./frontend.md) | `graphql-client` desde el schema |
| Contrato cruzado | [`xcheck`](../rustdoc/xcheck/index.html) | Leptos ≡ Node → resultado idéntico |

Los ejemplos se describen en los capítulos [Backend](./backend.md),
[Frontend](./frontend.md) y [Verificación](./verification.md); esta tabla es el
índice, con enlace directo al **código** en la [referencia de API
(rustdoc)](../rustdoc/api/index.html).

> La referencia rustdoc se genera junto al libro con `scripts/build-docs.sh`
> (mdBook + `cargo doc` en paralelo). Con `mdbook build` a secas, los enlaces
> `../rustdoc/…` no existen todavía.

## Tests implementados y cómo usarlos

La filosofía: **lógica pura testeada sin servicios externos**; lo que toca DB o
storage va tras `#[ignore]`; y el end-to-end se prueba por `curl` / script.

| Crate | Cubre | Notas |
|-------|-------|-------|
| `rust-gql-domain` | serde de `Role`; validación de `Email`/`UserId`/`UserName`; agregado `User` | corren en native **y** wasm |
| `api` (graphql) | `hello` resuelve; **paridad del SDL** con `user.gql`; stream de `count` | — |
| `api` (http) | POST `hello`, GET GraphiQL, **upload multipart** (jaydenseric), stream **SSE** | vía `tower::oneshot`, sin puerto ni DB |
| `api` (auth) | **vector HMAC real** de Better-Auth; parseo de cookie | prueba cross-lenguaje |
| `api` (db) | proyección `users`→`User`; rechazo de fila inválida | + `#[ignore]` contra DB real |
| `xtask` | diff de contrato: idéntico / campo quitado / tipo cambiado / aditivo / enum | 3 casos **negativos** |
| `rust-gql-frontend` | `config_parses` | binding de contrato: campo inexistente **no compila** |

Cómo ejecutarlos:

```bash
# por crate
cargo test                        # domain / frontend
cargo test --workspace            # backend (api + xtask + xcheck)

# integración real (necesita credenciales)
GQL_DATABASE__URL=... cargo test -p api -- --ignored          # sesión / repo
GQL_STORAGE__URL=... GQL_STORAGE__KEY=... cargo test -p api -- --ignored

# contrato cruzado (Leptos ≡ Node) contra el backend vivo
scripts/xcheck.sh

# binding negativo (debe FALLAR):
#   edita rust-gql-frontend/src/queries/hello.graphql a un campo inexistente
#   -> el build wasm falla ("No field named ... on Query")
```

## Sugerencias de benchmarking

Aún **no hay benchmarks** en el repo; estas son las vías recomendadas:

- **Micro (Rust)** — [criterion](https://crates.io/crates/criterion) para funciones
  puras: verificación HMAC de la cookie, proyección `users`→`User`, `build_schema()`.
- **Throughput/latencia GraphQL** — cargar `POST /graphql` con una query fija usando
  `oha`, `wrk`, `k6` o `vegeta`; comparar **Node vs Rust** en p50/p99, RPS y RSS.
- **DB** — [`#[sqlx::test]`](https://docs.rs/sqlx) con una BD de prueba efímera para
  medir consultas reales y el efecto del `DataLoader` (N+1 con y sin).
- **Resolvers** — las *extensions* de async-graphql (apollo-tracing) dan tiempos por
  resolver.
- **Qué medir** — arranque en frío / tamaño de binario, RSS bajo carga, p99, y el
  N+1 con/sin DataLoader.

> Honestidad de benchmarking: misma BD, cachés calientes, queries realistas y
> percentiles (no medias). No cherry-pickees el caso favorable.

