# Verificación

Todo lo documentado está comprobado por ejecución (no por suposición).

## `rust-gql-domain`

```bash
cd rust-gql-domain
cargo build                                   # native
cargo build --target wasm32-unknown-unknown   # wasm
cargo build --features graphql                # con derives async-graphql
cargo test                                    # 7 tests (Role serde + validación User/newtypes)
cargo clippy --all-targets -- -D warnings
```

## `rust-gql-backend`

```bash
cd rust-gql-backend
cargo build --workspace
cargo clippy --workspace --all-targets -- -D warnings

# fail-fast sin secreto -> error nombrando el campo
cargo run -p api

# sirve GraphQL en http://localhost:4000/graphql (POST) + GraphiQL (GET)
GQL_DATABASE__URL='postgres://user:pw@localhost:5432/gql' RUST_LOG=info cargo run -p api
# en otra terminal:
curl -s -X POST localhost:4000/graphql -H 'content-type: application/json' \
  -d '{"query":"{ hello }"}'   # -> {"data":{"hello":"Hello World!"}}

# subscription por SSE (stream de eventos):
curl -sN -X POST localhost:4000/graphql/sse -H 'content-type: application/json' \
  -d '{"query":"subscription { count(to: 3) }"}'   # -> 3x event: next

# xtask: emite el SDL y verifica el contrato contra los .gql del Node
cargo run -p xtask -- emit-schema   # -> schema.graphql
cargo run -p xtask -- diff          # exit 0 si no hay breaking ni staleness
```

## `rust-gql-frontend`

```bash
cd rust-gql-frontend
cargo build --target wasm32-unknown-unknown   # graphql-client valida HelloQuery vs schema.graphql
cargo test --lib                              # config_parses
cargo clippy --target wasm32-unknown-unknown --all-targets -- -D warnings
# binding de contrato: editar src/queries/hello.graphql a un campo inexistente
# -> el build wasm FALLA ("No field named ... on Query")
```

## `gofigeeks-gql-frontend/codegen` (tercer consumidor)

```bash
cd gofigeeks-gql-frontend/codegen
npm install
npm run generate   # genera generated/types.ts desde ../../rust-gql-backend/schema.graphql
# binding: una operación con un campo inexistente hace fallar `graphql-codegen`
```

## Ejecutar el cliente Leptos en el navegador (sin Trunk)

```bash
scripts/build-web.sh            # debug (incluye console_error_panic_hook)
scripts/build-web.sh --release  # producción lean (hook cfg-excluido)
# el CLI wasm-bindgen debe coincidir con la dep (el script lo valida y aborta si no)

cd rust-gql-frontend && python3 -m http.server 5173   # 5173 = origen permitido por CORS
# con el backend en :4000, abrir http://localhost:5173
# -> "hello: Hello World!" (fetch real) + Email::parse corriendo en wasm
```

## Contrato cruzado (Leptos ≡ Node)

```bash
scripts/xcheck.sh
# arranca el backend, ejecuta la misma operación `hello` desde el cliente Node
# (graphql-codegen) y desde el request Leptos (graphql_client, misma hello.graphql)
# -> PASS si el `data` es idéntico: {"hello":"Hello World!"}
```

## Este libro

```bash
# sólo el libro (por idioma)
mdbook build rust-gql-docs/es   # -> rust-gql-docs/book/es
mdbook build rust-gql-docs/en   # -> rust-gql-docs/book/en

# sitio completo: libro (ES/EN) + referencia API (rustdoc) en paralelo
scripts/build-docs.sh           # -> book/{es,en} + book/rustdoc
```

## Puerta global (todas las etapas)

```bash
cargo clippy --workspace --all-targets -- -D warnings && cargo fmt --all --check
```
