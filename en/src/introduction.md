# Introduction

> 🌐 **Language:** English · [Español](../es/introduction.html)

This book documents the migration of the **Taller GQL** GraphQL backend from
Node (GraphQL Yoga + Drizzle + Better-Auth) to a Rust workspace, serving the
**same contract** to two frontends: a new Leptos client and the existing Node
SPA, which keeps talking to the backend **with no query changes**.

The governing principle is separating **contract** from **implementation**:

- **Contract** — `schema.graphql` (emitted SDL) + standard HTTP semantics. A
  single artifact shared by async-graphql (Rust backend), graphql-client
  (Leptos) and graphql-codegen (Node SPA).
- **Implementation** — Rust crates. The shared domain crate is a Rust-side
  benefit; it **never crosses the wire** to Node.

Design consequence: the backend stays **client-agnostic**.

This book describes **what is already built and verified**, not what is planned.
