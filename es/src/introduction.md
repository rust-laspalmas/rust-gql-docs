# Introducción

> 🌐 **Idioma:** Español · [English](../en/introduction.html)

Este libro documenta la migración del backend GraphQL del **Taller GQL** desde
Node (GraphQL Yoga + Drizzle + Better-Auth) a un workspace Rust, sirviendo el
**mismo contrato** a dos frontends: un cliente Leptos nuevo y el SPA Node
existente, que sigue hablando con el backend **sin cambios de query**.

El principio rector es separar **contrato** de **implementación**:

- **Contrato** — `schema.graphql` (SDL emitido) + semánticas HTTP estándar. Único,
  compartido por async-graphql (backend Rust), graphql-client (Leptos) y
  graphql-codegen (SPA Node).
- **Implementación** — crates Rust. El crate de dominio compartido es un
  beneficio interno del lado Rust; **no cruza el cable** hacia Node.

Consecuencia de diseño: el backend queda **agnóstico al cliente**.

Este libro describe **lo que ya está montado y verificado**, no lo planeado.
