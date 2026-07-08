# Backend (`rust-gql-backend`)

Workspace con `resolver = "3"` y tres miembros: **`crates/api`** (lib + bin
`gql-api`), **`xtask`** (contrato) y **`xcheck`** (contrato cruzado). Las versiones
de dependencias viven en un único `[workspace.dependencies]` (no se repiten entre
crates).

## Schema y contrato

La lib de `api` define el **schema code-first** con async-graphql y expone
`build_schema()` (lo usa `xtask` para emitir el SDL):

- Root `Query { hello }`.
- Proyección GraphQL de `User` — campos `String` desde los newtypes del dominio,
  camelCase automático.
- Scalar propio **`DateTimeISO`** (RFC3339) que casa con el de graphql-scalars del
  Node.
- `User` y `Role` se registran explícitamente (`register_output_type`): el contrato
  Node los declara aunque ninguna query los devuelva aún.

## Datos y DataLoader

Módulo `db`:

- `UserRepository` con **sqlx** contra el Postgres existente.
- Proyecta la tabla `users` (sólo las **7 columnas del contrato**, validadas vía
  los newtypes del dominio) al `type User`.
- `find_by_ids` usa `id = ANY($1)` — **una query para N claves**.
- `loader::UserLoader` lo envuelve en un `DataLoader` de async-graphql (**anti-N+1**).

## Transporte y CORS

Módulo `http`:

| Ruta | Sirve |
|------|-------|
| `POST /graphql` | operaciones (async-graphql-axum) |
| `GET /graphql` | GraphiQL IDE |
| `GET/POST /graphql/sse` | subscriptions por SSE |
| `GET /graphql/ws` | subscriptions graphql-ws |

CORS de tower-http con orígenes y credenciales tomados de `[cors]` (cookies de
sesión ⇒ **sin wildcard**).

## Auth de sesión

Módulo `auth`: `trait AuthProvider` + `SessionAuthProvider`.

- Valida la cookie `better-auth.session_token`: firma HMAC-SHA256 **opcional**
  (según `[auth.session].verify_signature`) + lookup en `sessions`.
- Produce un `Principal`, expuesto a la vez en el **contexto GraphQL** y como
  **`Extension` axum**.
- El contrato de la cookie está **verificado contra `better-auth@1.6.19`**.

## Storage y uploads

Módulo `storage`: `trait Storage` + `SupabaseStorage`. Respalda la mutation
`uploadMedia(file: Upload!)`, que recibe ficheros por la spec **multipart
jaydenseric** (compatible con apollo-upload-client, parseada por
async-graphql-axum) y los sube al bucket de `[storage]`.

## Subscriptions

`SubscriptionRoot { count }`, servido por **dos transportes seleccionables por
feature**:

- `subs-sse` — adaptador SSE, compat graphql-sse (el SPA Node lo consume sin
  WebSocket).
- `subs-ws` — graphql-ws nativo de async-graphql-axum.

## Features

Ortogonales, seleccionables por feature/config:

| Grupo | Opciones |
|-------|----------|
| Auth | `auth-session` · `auth-jwt` · `auth-oidc` |
| Subscriptions | `subs-sse` · `subs-ws` |
| Storage | `storage-supabase` · `storage-s3` |

## `xtask` — contrato

- `emit-schema` escribe `schema.graphql` desde `build_schema().sdl()`.
- `diff` parsea a AST el SDL emitido y los `.gql` del Node y compara un modelo
  canónico (tipo→campos, enum→variantes), ignorando descripciones, `schema{}`,
  directivas built-in y scalars huérfanos. Falla ante cambios *breaking*
  (quitar/cambiar tipo o campo) o si el `schema.graphql` commiteado está
  desactualizado; los cambios aditivos son informativos.
