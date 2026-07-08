# Shared domain (`rust-gql-domain`)

A `lib` crate that compiles to **native and `wasm32`**. It is the crate that
**crosses the boundary**: the backend uses it with the `graphql` feature
(async-graphql derives); the frontend without it, so **async-graphql never enters
the wasm bundle**.

## What it holds

- **`enum Role`** — mirror of `role.gql`; serde always, async-graphql derives only
  under the `graphql` feature.

- **Newtypes validated with `garde`**, each with a `parse()` that validates on
  construction:

  | Type | Rule |
  |------|------|
  | `UserId` | non-empty string (Better-Auth `text` id, **not** a UUID) |
  | `Email` | email format |
  | `UserName` | length |

- **The `User` aggregate** — mirror of `type User` (id, email, name, emailVerified,
  image, biography, createdAt), with nested (`dive`) validation and `image`
  validated as a URL when present.

## Why it matters

The **domain validation runs identically** on the backend (native) and in the
browser (wasm) — a single source compiled to two targets. The GraphQL projection of
`User` (`String` fields, per the Node contract) is assembled in `api`, not here.
