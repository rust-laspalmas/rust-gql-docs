<a href="https://graphql.org/"><img src="imgs/graphql.svg" align="left" width="250" alt="GraphQL"/></a>

# rust-gql-docs

> Bilingual **mdBook** (ES/EN) + **rustdoc** — the why of the GraphQL-in-Rust migration.

<br clear="left"/>

Español: [README_ES.md](./README_ES.md)

Documents the port of the GofiGeeks GraphQL workshop to Rust: the governing
principle (a single `schema.graphql` checked by the compiler on both sides), the
architecture, the dependency mapping, and how everything is verified. The book links
code items to their source in the generated API reference (rustdoc).

## Layout

- `es/`, `en/` — the two mdBooks (each with its own `book.toml` + `src/`).
- `book/` — build output: `index.html` (language portal), `es/`, `en/`, `rustdoc/`.

## Requirements

- [`mdbook`](https://rust-lang.github.io/mdBook/) (`cargo install mdbook`)
- Rust **1.95+** with `cargo` (for the rustdoc part)

## Build

```bash
# book only (per language)
mdbook build es      # -> book/es
mdbook build en      # -> book/en

# full site: book (ES/EN) + API reference (rustdoc) in parallel
scripts/build-docs.sh     # from the workspace root -> book/{es,en,rustdoc}

# serve
python3 -m http.server 3000 --directory book   # open http://localhost:3000
```

## The ecosystem

This repo **documents** the code repos:

| Repo | What it contributes |
|------|---------------------|
| [rust-gql-domain](https://github.com/rust-laspalmas/rust-gql-domain) | shared newtypes + validation |
| [rust-gql-backend](https://github.com/rust-laspalmas/rust-gql-backend) | the server; emits `schema.graphql` |
| [rust-gql-frontend](https://github.com/rust-laspalmas/rust-gql-frontend) | the Leptos wasm client |
| **rust-gql-docs** (this) | the mdBook + rustdoc that explain them |

---

<a href="https://rust-laspalmas.dev/"><img src="imgs/rust-laspalmas.svg" align="left" width="150" alt="Rust Las Palmas"/></a>

<br>

Part of the **GofiGeeks GraphQL → Rust** learning exploration by
[Rust Las Palmas](https://rust-laspalmas.dev) · [jesusperez.pro](https://jesusperez.pro).

Not a fork of the workshop — a companion.

<br clear="left"/>
