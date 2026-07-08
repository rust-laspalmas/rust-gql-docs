# Qué se ha montado

Tres carpetas hermanas bajo `gql/`, más el SPA Node intacto:

```
gql/
├── rust-gql-domain/     # crate PURO compartido (native + wasm32)
├── rust-gql-backend/    # workspace: crates/api (bin) + xtask + xcheck
├── rust-gql-frontend/   # crate Leptos CSR (wasm32)
├── rust-gql-docs/       # este mdBook
└── gofigeeks-gql-frontend/   # SPA Node — INTACTO
```

El eje de todo es un **único `schema.graphql`**, emitido por el backend y
consumido por tres clientes:

| Consumidor | Rol |
|------------|-----|
| **async-graphql** | el backend Rust — lo **emite** |
| **graphql-client** | el cliente Leptos — compila contra él |
| **graphql-codegen** | el SPA Node — genera tipos de él |

Cada pieza tiene su página:

- [Dominio compartido](./domain.md) — newtypes + validación (native **y** wasm)
- [Backend](./backend.md) — schema, datos, transporte, auth, storage, subscriptions
- [Frontend Leptos](./frontend.md) — cliente wasm con el contrato verificado en compilación
