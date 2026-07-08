# Dominio compartido (`rust-gql-domain`)

Crate `lib` que compila a **native y `wasm32`**. Es el crate que **cruza la
frontera**: el backend lo usa con la feature `graphql` (derives de async-graphql);
el frontend sin ella, de modo que **async-graphql nunca entra en el bundle wasm**.

## Qué contiene

- **`enum Role`** — espejo de `role.gql`; serde siempre, derives de async-graphql
  sólo bajo la feature `graphql`.

- **Newtypes validados con `garde`**, cada uno con `parse()` que valida al
  construir:

  | Tipo | Regla |
  |------|-------|
  | `UserId` | string no vacío (id `text` de Better-Auth, **no UUID**) |
  | `Email` | formato de email |
  | `UserName` | longitud |

- **Agregado `User`** — espejo de `type User` (id, email, name, emailVerified,
  image, biography, createdAt), con validación anidada (`dive`) e `image` validado
  como URL cuando está presente.

## Por qué importa

La **validación de dominio corre idéntica** en el backend (native) y en el
navegador (wasm) — una sola fuente compilada a dos targets. La proyección GraphQL
de `User` (campos `String`, según el contrato Node) se ensambla en el `api`, no
aquí.
